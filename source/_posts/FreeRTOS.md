---
title : FreeRTOS
date : 2022-01-25 00:01:230
tag : [esp32,c,freertos]
categories : [硬件]
description : 基于Michael_ee老师esp-idf上实时操作系统FreeRTOS的笔记，等待同步中。。。
---
# FreeRTOS

</style>
<div id="bilibili" >
    <iframe width="600" height="400" src="//player.bilibili.com/player.html?aid=634784415&bvid=BV1Nb4y1q7xz&cid=461277523&page=1" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true"> </iframe>
</div>

[https://www.freertos.org/](https://www.freertos.org/)

💼 [FreeRTOS参考指南](https://oneindex.yyin.top/share/pdf/freertos/FreeRTOS_Reference_Manual_V10.0.0.pdf)

📁[FreeRTOS 官方教程](https://oneindex.yyin.top/share/pdf/freertos/Mastering_the_FreeRTOS_Real_Time_Kernel-A_Hands-On_Tutorial_Guide.pdf)

🧭 [ESP-IDF 官方手册](https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32c3/get-started/index.html)

在嵌入式领域中，嵌入式实时操作系统正得到越来越广泛的应用。采用嵌入式实时操作系统(RTOS)可以更合理、更有效地利用CPU的资源，简化应用软件的设计，缩短系统开发时间，更好地保证系统的实时性和可靠性。

函数的命名规则

```C
u :代表unsigned。
s :代表short。
c :char。
uc，us：unsigned char，unsigned short，分别对应uint8_t,uint16_t。
x :为用户自定义的数据类型，比如结构体，队列等。
ux：unsigned且用户自定义的类型。需要注意的是size_t变量前缀也是ux。
e :枚举变量
p :指针变量 类似(uint16_t *)变量前缀为pus
prv :static函数
v: void函数
```

# 一、简介

## 实时操作系统

The merit of Real-Time Operation System

1. 可以把功能分成模块，团队合作提高工作效率
2. 可以分模块测试，单独模块测试，然后进行系统测试
3. 复用代码，只需要改变相应的接口，就可以用于不同的项目
4. 可以把不同的任务，按照不同的优先级别，把实时性要求高的任务优先执行（比如说中断，我们需要马上相应并执行相应的程序）实时性较低的任务（比如说lcd屏幕显示）赋予较低的优先级别
5. 小而简单，只有三个核心C文件，极其适合嵌入式项目
6. 广泛移植

## 基于ESP32的FreeRTOS

1. 集成Wi-Fi和Bluetooth
2. 内部flash和SRAM
3. I2C，SPI，I2S、UART、ADC、DAC、RTC，USB外设支持
4. 极小的尺寸，内部集成晶振
5. 价格美丽

# 二、系统启动流程

1. [一级引导程序](https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32c3/api-guides/startup.html#first-stage-bootloader) 被固化在了 ESP32-C3 内部的 ROM 中，它会从 flash 的 0x0 偏移地址处加载二级引导程序至 RAM (IRAM & DRAM) 中。
2. [二级引导程序](https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32c3/api-guides/startup.html#second-stage-bootloader) 从 flash 中加载分区表和主程序镜像至内存中，主程序中包含了 RAM 段和通过 flash 高速缓存映射的只读段。
3. [应用程序启动阶段](https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32c3/api-guides/startup.html#application-startup) 运行，这时第二个 CPU 和 RTOS 的调度器启动。(应用程序执行入口)

bootloader源码目录：`esp\esp_idf\components\bootloader_support\src`

freertos源码目录：`esp\esp_idf\components\freertos\src`

`\port` 针对不同的系统架构移植的代码

应用程序入口:

freertos目录下的启动流程

`app_main()`：主任务（优先级比最小值高）

←`main_task()`：设置堆栈,定义main_task()

←`esp_startup_start_app_common()`：初始化main_task()

←`esp_startup_start_app()`:创建main_task(),并且按照不同的任务优先度进行调度操作 `freertos\port.c`

esp_system下的启动流程：针对芯片的特定代码

←`start_cup0_default()`:核心的组件和服务 `esp_system\startup.c`

←`start_cup0`:系统初始化

←`g_startup_fn_t[]`:调动不同的cpu初始化程序

←`SYS_STARTUP_FN()`:宏

←`call_start_cpu0()`:初始化C运行环境，对SoC硬件进行初始化配置 `esp_system\port\cpu_static.c`

←`esp_system\ld\esp32c3\section.ld.in` 链接文件，application startup的最终入口

# 三、Task的创建和删除

## 任务创建

```C
/* 调用xTaskCreate()来创建一个task*/
TaskHandle_t xTaskCreateStatic( TaskFunction_t pvTaskCode,   //一个简单地指针指向要创建的task函数(就是函数名)
             const char * const pcName,                      //任务名称
             uint32_t ulStackDepth,                          //表示分配多少内存给该task，1024 = 1k
             void *pvParameters,                             //task的传递参数，该参数会由xTaskCreate()传递到myTask()当中
             UBaseType_t uxPriority,                         //task的优先级
             StackType_t * const puxStackBuffer,
             StaticTask_t * const pxTaskBuffer );
```

## 任务删除

```C
void vTaskDelete( TaskHandle_t pxTask );    //任务的句柄
```

```C
#include <stdio.h>
#include "sdkconfig.h"
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_system.h"
#include "esp_spi_flash.h"
//------------------------------------------------------------------------------------
//创建task主体函数
//传入一个void 指针型变量，用这种方式可以强制转换任何其他类型，而传入task函数中
void myTask(void *pvParam){
    // for( ;; )
    {
        printf("hello_world! First!\n");
        vTaskDelay(pdMS_TO_TICKS(1000));

        printf("hello_world! Second!\n");
        vTaskDelay(pdMS_TO_TICKS(1000));

        printf("hello_world! Third!\n");
        vTaskDelay(pdMS_TO_TICKS(1000));
    }

    //为空则删除当前的task
    vTaskDelete(NULL);
}
//------------------------------------------------------------------------------------

void app_main(void)
{
    TaskHandle_t myHandler = NULL;
    
    xTaskCreate(myTask,"myTask1",1024,NULL,1,NULL);

    // vTaskDelay(pdMS_TO_TICKS(8000));

    // //判断句柄是否为空
    // if(myHandler != NULL){
    //     vTaskDelete(myHandler);
    // }
}

```

# 四、Task的传值

## 常见数据类型

```C
#include <stdio.h>
#include "sdkconfig.h"
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_system.h"
#include "esp_spi_flash.h"
//------------------------------------------------------------------------------------
void myTask(void *pvParam){
    
    int *pInt;
    pInt = (int *)pvParam;

    printf("I got testNum= %d\n",*pInt);
    vTaskDelay(pdMS_TO_TICKS(1000));

    vTaskDelete(NULL);
}
int testNum = 2;
//------------------------------------------------------------------------------------

void app_main(void)
{
    xTaskCreate(myTask,"myTask1",2048,(void *)&testNum,1,NULL);
}
```

## 数组

```C
#include <stdio.h>
#include "sdkconfig.h"
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_system.h"
#include "esp_spi_flash.h"
//------------------------------------------------------------------------------------
void myTask(void *pvParam){
    
    int *pInt;
    pInt = (int *)pvParam;

    printf("I got testNum= %d\n",*pInt);
    vTaskDelay(pdMS_TO_TICKS(1000));

    vTaskDelete(NULL);
}
int testNum = 2;
//------------------------------------------------------------------------------------

void app_main(void)
{
    xTaskCreate(myTask,"myTask1",2048,(void *)&testNum,1,NULL);
}
```

## [结构体](https://www.wolai.com/dNG4Y93jibfT9ENrXVmRY1)

```C
#include <stdio.h>
#include "sdkconfig.h"
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_system.h"
#include "esp_spi_flash.h"
typedef struct A_STRUCT
{
    int iMem1;
    int iMem2;
}xStruct;

xStruct xStructTest =  {6,9};
//------------------------------------------------------------------------------------
void myTask(void *pvParam)
{

    xStruct *pStrTest = (xStruct *)pvParam;

    printf("I got struct iMem1 = %d\n", pStrTest->iMem1);
    printf("I got struct iMem2 = %d\n", pStrTest->iMem2);

    vTaskDelay(pdMS_TO_TICKS(1000));

    vTaskDelete(NULL);
}
//------------------------------------------------------------------------------------
void app_main(void)
{
    xTaskCreate(myTask, "myTask1", 2048, (void *)testArr, 1, NULL);
}

```

注意：定义的结构体如果是指针，访问成员时就用->；如果定义的是结构体变量，访问成员时就用.

## [字符串](http://c.biancheng.net/view/1833.html)

```C
#include <stdio.h>
#include "sdkconfig.h"
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_system.h"
#include "esp_spi_flash.h"
static  const char *pText = " I am Penguin";
//------------------------------------------------------------------------------------
void myTask(void *pvParam)
{         
    char *pStrTest = (char *)pvParam;
    printf("I got Message =%s \n",pStrTest);
    vTaskDelay(pdMS_TO_TICKS(1000));
    vTaskDelete(NULL);
}

//------------------------------------------------------------------------------------
void app_main(void)
{
    xTaskCreate(myTask, "myTask1", 4096, (void *)pText, 1, NULL);
}

```

ps: 在输出字符串的时候要加`\n`,否侧可能会出现重启，字符串不输出等现象，可能原因是堆栈设置过小

# 五、Task的优先级别

## 优先级别的定义

每一个任务被分配按照由 0 到 ( configMAX_PRIORITIES - 1 )的优先级,而 configMAX_PRIORITIES 这个宏定义存在于 FreeRTOSConfig.h.

directory for esp32c3 : `esp\esp_idf\components\freertos\port\xtensa\include\freertos`

被定义的任务，分配的优先级小于configMAX_PRIORITIES-1，自定义大于configMAX_PRIORITIES-1的数值仍旧按照configMAX_PRIORITIES - 1定义

如果修改了 FreeRTOSConfig.h这个文件，将会重新定义400多个文件

### 获取优先级

```C
UBaseType_t uxTaskPriorityGet( TaskHandle_t pxTask );
```

```C
#include <stdio.h>
#include "sdkconfig.h"
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_system.h"
#include "esp_spi_flash.h"
static const char *pText = "task priority test";
//------------------------------------------------------------------------------------
void myTask(void *pvParam)
{
    char *pStrTest = (char *)pvParam;
    printf("I got Message =%s \n", pStrTest);
    vTaskDelay(pdMS_TO_TICKS(10000));
    vTaskDelete(NULL);
}
//------------------------------------------------------------------------------------
void app_main(void)
{
    UBaseType_t iPriority = 0;
    TaskHandle_t pxTask = NULL;
    //句柄表示指针的指针，此处传递创建的空句柄要写取指符号&
    xTaskCreate(myTask, "myTask1", 4096, (void *)pText, 265, &pxTask);
    //获取当前任务的优先级
    iPriority = uxTaskPriorityGet(pxTask);
    printf("iPriority = %d \n", iPriority);
}
```

## 同样的优先级的执行

如果有同等优先级的任务，它们会基于切片循环调度方案（time sliced round robin scheduling scheme.）以共享可用的处理时间。谁先创建谁先运行。

```C
#include <stdio.h>
#include "sdkconfig.h"
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_system.h"
#include "esp_spi_flash.h"

//------------------------------------------------------------------------------------
void myTask1(void *pvParam)
{
    for( ; ; )
    {
        printf("task1-111\n");
        vTaskDelay(pdMS_TO_TICKS(1000));
    }
}
void myTask2(void *pvParam)
{
    for( ; ; )
    {
        printf("task2-222\n");
        vTaskDelay(pdMS_TO_TICKS(1000));
    }
}
//------------------------------------------------------------------------------------
void app_main(void)
{
    xTaskCreate(myTask1, "myTask1", 1024, NULL, 1, NULL);
    xTaskCreate(myTask2, "myTask2", 1024, NULL, 1, NULL);    
}

```

```C
task1-111
task2-222
task2-222
task1-111
task1-111
task2-222
task2-222
task1-111
task1-111

```

## 不同的优先级别的执行

系统会默认运行高优先级别的task

```C
void app_main(void)
{
    xTaskCreate(myTask1, "myTask1", 1024, NULL,1, NULL);
    xTaskCreate(myTask2, "myTask2", 1024, NULL,3, NULL);
}

```

```C
task2-222
task1-111
task2-222
task1-111
task2-222
task1-111
task2-222
```

## 优先级反转(修改优先级)

```C
void vTaskPrioritySet( TaskHandle_t pxTask, UBaseType_t uxNewPriority );
```

```C
void app_main(void)
{
    TaskHandle_t pxTask = NULL;

    xTaskCreate(myTask1, "myTask1", 1024, NULL,1, &pxTask);
    xTaskCreate(myTask2, "myTask2", 1024, NULL,3, NULL);
    
    vTaskPrioritySet(pxTask,3);
    printf("priority changed-->");

}

```

```C
task2-222
task1-111
priority changed to 3--> task1-111
task2-222
task1-111
task2-222

```

# 六、Task的挂起和恢复

## 任务的状态

- **Running**
 当一个任务Running时，我们说这个任务正处于运行状态. 在这个阶段任务将会利用处理器(processor). 如果运行RTOS的处理器只有一个核，那么在任何给定的时间里，只能有一个任务处于running的状态。
- **Ready**
 Ready 任务是指那些能够执行的任务(它们不是处于Blocked或者Suspended状态) ，但目前还没有执行，因为有一个同等或者更高优先级的任务已经出于Running状态。
- **Blocked**
 如果一个任务当前正在等待一个临时或外部事件，则该任务被称为处于Blocked状态。 例如，如果一个任务调用了vTaskDelay()，它将被阻塞(被置于Blocked状态)，直到延迟时间过期——这是一个临时事件。 任务也可以阻塞等待队列、信号量、事件组、通知或信号量事件。 处于Blocked状态的任务通常有一个“超时”时间，在此之后，任务将被超时，并且被解除阻塞，即使任务等待的事件没有发生。  
- **Suspended**
 处于“挂起”状态的任务与处于“阻塞”状态的任务一样，不能选择进入“运行”状态的任务，但处于“暂停”状态的任务没有超时时间。 相反，任务只有在分别通过vTaskSuspend()和xTaskResume() API调用显式地命令进入或退出Suspended状态时才会进入或退出。  

![](https://qiniu.yyin.top/tskstate.gif)

## 任务挂起

```C
void vTaskSuspend( TaskHandle_t pxTaskToSuspend );
```

### 外部挂起

```C
void app_main(void)
{
    TaskHandle_t pxTask = NULL;

    xTaskCreate(myTask1, "myTask1", 1024, NULL,1, &pxTask);
    xTaskCreate(myTask2, "myTask2", 1024, NULL,2, NULL);
    
    vTaskDelay(2000 / portTICK_PERIOD_MS);
    vTaskSuspend(pxTask);
    printf("<<<<<<<<<<<Task1 suspended>>>>>>>>>\n");
}
```

```C
task2-222
task1-111
task2-222
task1-111
task2-222
task1-111
<<<<<<<<<<<Task1 suspended>>>>>>>>>
task2-222
task2-222
task2-222
task2-222
task2-222
```

### 内部挂起

```C
void myTask1(void *pvParam)
{
    for (;;)
    {
        printf("task1-111\n");
        vTaskDelay(pdMS_TO_TICKS(1000));
    
        printf("<<<<<<<<<<<Task1 suspended>>>>>>>>>\n");
        vTaskSuspend(NULL);
    }
}
```

```C
task2-222
task1-111
task2-222
<<<<<<<<<<<Task1 suspended>>>>>>>>>
task2-222
task2-222
task2-222
task2-222
```

## 任务恢复

```C
void app_main(void)
{
    TaskHandle_t pxTask = NULL;

    xTaskCreate(myTask1, "myTask1", 1024, NULL, 1, &pxTask);
    xTaskCreate(myTask2, "myTask2", 1024, NULL, 2, NULL);
    //暂停调度器对task1的调度
    vTaskSuspend(pxTask);
    printf("<<<<<<<<<<<Task1 suspended>>>>>>>>>\n");

    vTaskDelay(2000 / portTICK_PERIOD_MS);
    //使得task1参与调度器的调度
    printf("<<<<<<<<<<<Task1 Resumed>>>>>>>>>\n");
    vTaskResume(pxTask);
}
```

```C
task2-222
task1-111
<<<<<<<<<<<Task1 suspended>>>>>>>>>
task2-222
task2-222
task1-111
<<<<<<<<<<<Task1 Resumed>>>>>>>>>>>
task2-222
task1-111
```

## 全部挂起和全部恢复

```C
void vTaskResume( TaskHandle_t pxTaskToResume );
void vTaskSuspendAll( void );

```

注意：在任务全部挂起的时候不可以调用其他FreeRTOS的 API，但可以调用自己设计的函数。还注意系统里面的看门狗，当一个任务执行太长的时间而没有喂狗的话，任务会堵塞重启

我们创建并运行两个task，因为task1会运行比较长的时间，所以此时任务调度器会进行调度，当task1时间片到达的时候，会调度到task2打印出相关的信息。如果在test begin和task end之间出现task2 的内容就证明两个任务在进行切换

面对这样的场景：假设我们运行一段代码，这个代码不想被其他任务切换，这是一段与时间紧密相关的代码。则我们使用`vTaskSuspendAll() ... vTaskResumeAll()`方式将这段代码包裹起来，注意在这之间不可以使用其他FreeRTOS相关API

ps：for循环99999次会触发看门狗

```C
#include <stdio.h>
#include "sdkconfig.h"
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_system.h"
#include "esp_spi_flash.h"

//------------------------------------------------------------------------------------
void myTask1(void *pvParam)
{
    printf("<<<<<Test begin>>>>>\n");
    vTaskSuspendAll();
    for (int i = 0; i < 9999; i++)
    {
        for (int j = 0; j < 9999; j++)
        {
            /* 空循环，占用cpu的处理时间，模拟计算 */
        }
        
    }
    xTaskResumeAll();
    printf("<<<<<Test end>>>>>\n");

    vTaskDelete(NULL);
}
void myTask2(void *pvParam)
{
    for (;;)
    {
        printf("task2-222\n");
        vTaskDelay(pdMS_TO_TICKS(1000));
    }
}
//------------------------------------------------------------------------------------
void app_main(void)
{

    TaskHandle_t pxTask = NULL;

    xTaskCreate(myTask1, "myTask1", 1024, NULL, 1, &pxTask);
    xTaskCreate(myTask2, "myTask2", 1024, NULL, 1, NULL);

    
}
```

# 七、Task系统信息显示

```C
void vTaskList( char *pcWriteBuffer ); //输入参数是一个指向Buffer的指针
```

这个函数可以打印出系统里面所有任务的状态、优先级、堆栈、ID

![调用vTaskList()生成的内容试例](https://qiniu.yyin.top/20220124223501.png)

这个函数可以方便我们进行调试

|简写|状态|
|---|---|
|X|Executing |
|B|Blocked|
|R|Ready|
|S|Suspended|
|D|Delete|

对于这个函数的使用，需要在打开`FreeRTOSConfig.h`中的两个宏定义

`configUSE_TRACE_FACILITY`

`configUSE_STATS_FORMATTING_FUNCTIONS`

我们可以在`menuconfig`中进行设置

![](https://qiniu.yyin.top/20220124230841.png)

```C
#include <stdio.h>
#include "sdkconfig.h"
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_system.h"
#include "esp_spi_flash.h"

//------------------------------------------------------------------------------------
void myTask1(void *pvParam)
{
    printf("<<<<<Test begin>>>>>\n");
    vTaskSuspendAll();
    for (int i = 0; i < 9999; i++)
    {
        for (int j = 0; j < 9999; j++)
        {
            /* 空循环，占用cpu的处理时间，模拟计算 */
        }
        
    }
    xTaskResumeAll();
    printf("<<<<<Test end>>>>>\n");

    vTaskDelete(NULL);
}
void myTask2(void *pvParam)
{
    for (;;)
    {
        printf("task2-222\n");
        vTaskDelay(pdMS_TO_TICKS(1000));
    }
}
//------------------------------------------------------------------------------------
void app_main(void)
{

    TaskHandle_t pxTask = NULL;

    xTaskCreate(myTask1, "myTask1", 1024, NULL, 1, &pxTask);
    xTaskCreate(myTask2, "myTask2", 1024, NULL, 1, NULL);

    static char pcWriteBuffer[512] = {0};
    
    vTaskList(pcWriteBuffer);

    printf("--------------------------------------------\n");
    printf("Name          State   Priority  Stack   Num \n");
    printf("%s\n",pcWriteBuffer);
}

```

```C
--------------------------------------------
Name          State   Priority  Stack   Num 
myTask2         R       1       728     5
main            X       1       3712    2
myTask1         R       1       732     4
IDLE            R       0       2004    3
esp_timer       S       22      3756    1
```

ps:此处如果出现找不到vTaskList(),可以尝试clean一下，或者在ESP-EDF终端输入`idf.py clean`

Stack ：这是一个在任务整个生命周期当中最小数量的可用内存

# 八、Task堆栈设置和调试

之前我们设置了一个缺省的值来设置任务内存，我们可以调用`uxTaskGetStackHighWaterMark()`来获取运行过程中栈中剩余的空间，这就是所谓"High Water Mark",当我们打印出这个值后我们可以估计在整个任务运行过程中会有内存剩下，我们就可以相应的减少或者增加栈的空间。`vTaskList()`同样可以打印出这样的值，但是对于嵌入式系统来说，内部的资源很有限，调用`vTaskList()`会消耗大量资源，所以我们可以使用`uxTaskGetStackHighWaterMark()`，取得我们关心的堆栈值的变化。

```C
UBaseType_t uxTaskGetStackHighWaterMark(TaskHandle_t xTask);
```

该函数的返回值是栈空间还剩余的最小空间，如果这个值越接近0的话，越接近堆栈溢出，这就会导致reset重启。

```C
#include <stdio.h>
#include "sdkconfig.h"
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_system.h"
#include "esp_spi_flash.h"

//------------------------------------------------------------------------------------
void myTask1(void *pvParam)
{
    for (;;)
    {
        printf("task1\n");
        vTaskDelay(1000 / portTICK_PERIOD_MS);
    }
}
//------------------------------------------------------------------------------------
void app_main(void)
{

    TaskHandle_t pxTask = NULL;

    xTaskCreate(myTask1, "myTask1", 1024, NULL, 1, &pxTask);

    UBaseType_t iStack;

    while (1)
    {
        iStack = uxTaskGetStackHighWaterMark(pxTask);
        printf("task iStack = %d \n",iStack);
        vTaskDelay(3000 / portTICK_PERIOD_MS);
    }
}

```

```C
task iStack = 732
task1
task1
task1
task1
task iStack = 408 
task1
```

我们可以利用这样的方法调试并设置任务的空间大小，同时可以估算一个函数的大小，这是非常好用的设计思想。

# 九、[Task看门狗](https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32c3/api-reference/system/wdts.html)

## 中断看门狗Interrupt watchdog

中断看门狗确保 FreeRTOS 任务切换到中断后不会被长时间阻塞。如果长时间阻塞，就意味着没有其他任务参与，包括潜在的重要任务，如 WiFi 任务和idle任务， 这些无法任务将获得CPU的 运行。当程序运行陷入死循环，而中断禁用或挂起中断时，阻塞的任务就可能触发中断。

1.中断被触发后，就可以让我们利用OpenOCD或者gdbstub来调试，告诉我们开发人员什么触发了中断的watchDog，即触发中断的原因，让我梦改正代码中的错误。

2.在生产环境中，当不可知的一些条件触发了中断看门狗，我们会在中断看门狗中重启我们的系统，从而使我们从错误中恢复过来。提高产品的稳定性。

这个看门狗调用的是定时器组1(timer group 1)

对于中断看门狗需要配置[CONFIG_ESP_INT_WDT](https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32c3/api-reference/kconfig.html#config-esp-int-wdt)和

[CONFIG_ESP_INT_WDT_TIMEOUT_MS](https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32c3/api-reference/kconfig.html#config-esp-int-wdt-timeout-ms).这两个宏定义

![](https://qiniu.yyin.top/20220125011006.png)

## 任务看门狗Task Watchdog Timer

Task Watchdog 是针对我们的任务的看门狗，有一些任务需要周期性运行，比如说idle任务，我们默认设置的是5秒，因为有其他的关键任务是依赖于这个idle任务而运行的。如果在5秒以内这个任务没有被运行就会触发任务看门狗。

任务看门狗会打印相关的信息以供调试，它也可以重启我们的系统。它有一个用户的callback函数，我们可以在这个函数里面重启我们的系统，使得整个系统有着更加强大的健壮性，使得我们的产品可以从未知的任务中恢复过来。

[**函数详情**](https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32c3/api-reference/system/wdts.html#functions)

|函数|简述|
|---|---|
|void `esp_int_wdt_init`(void)|初始化一个看门狗|
|[esp_err_t](https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32c3/api-reference/system/esp_err.html#_CPPv49esp_err_t)`esp_task_wdt_deinit`(void）|取消看门狗|
|[esp_err_t](https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32c3/api-reference/system/esp_err.html#_CPPv49esp_err_t) `esp_task_wdt_add`([TaskHandle_t](https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32c3/api-reference/system/freertos.html#_CPPv412TaskHandle_t) *handle*)|在任务中添加看门狗|
|[esp_err_t](https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32c3/api-reference/system/esp_err.html#_CPPv49esp_err_t) `esp_task_wdt_reset`(void)|任务看门狗的重置,喂狗|
|[esp_err_t](https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32c3/api-reference/system/esp_err.html#_CPPv49esp_err_t) `esp_task_wdt_delete`([TaskHandle_t](https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32c3/api-reference/system/freertos.html#_CPPv412TaskHandle_t) *handle*)|任务看门狗的删除|
|[esp_err_t](https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32c3/api-reference/system/esp_err.html#_CPPv49esp_err_t) `esp_task_wdt_status`([TaskHandle_t](https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32c3/api-reference/system/freertos.html#_CPPv412TaskHandle_t) *handle*)|取得任务看门狗的状态|

默认情况下系统之监控idle任务，如果我们想让自定义的任务也具有相关周期性运行的特点，如果在相应的周期内该自定义的任务没有执行，则提示或者进行相应的处理,那么我们就需要在我们的任务中调用`esp_task_wdt_add()`，将自定义任务添加到任务看门狗的监控列表。

调用以上函数，需要头文件`components/esp_system/include/esp_task_wdt.h`

### 触发IDLE任务的看门狗

```C
#include <stdio.h>
#include "sdkconfig.h"
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_system.h"
#include "esp_spi_flash.h"

//------------------------------------------------------------------------------------

void myTask1(void *pvParam)
{
   while(1){
   //如何预防触发看门狗？
   /*1.调用阻塞函数，让低优先级的函数得以运行例如
       vTaskDelay(1000/portTICK_PERIOD_MS)
    */
   }
}
//------------------------------------------------------------------------------------
void app_main(void)
{
    TaskHandle_t pxTask = NULL;
    /*2.改变当前任务的优先级别，当任务有相同的优先级的时候，任务调度器会切换相同的时间片给俩个任务
        从而保证两个任务都可以得到运行
        xTaskCreate(myTask1, "myTask1", 1024, NULL, 0, &pxTask);
     */
    xTaskCreate(myTask1, "myTask1", 1024, NULL, 1, &pxTask);
}

```

```C
E (10261) task_wdt: Task watchdog got triggered. The following tasks did not reset the watchdog in time:
E (10261) task_wdt:  - IDLE (CPU 0)
E (10261) task_wdt: Tasks currently running:
E (10261) task_wdt: CPU 0: myTask1
```

### 让自定义任务被任务看门狗监视

```C
#include <stdio.h>
#include "sdkconfig.h"
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_system.h"
#include "esp_spi_flash.h"

#include "esp_task_wdt.h"  //新添加的头文件
//------------------------------------------------------------------------------------

void myTask1(void *pvParam)
{
    while(1){
        printf("task1\n");
        vTaskDelay(1000 / portTICK_PERIOD_MS);
    }
}
void myTask2(void *pvParam)
{
    //将当前任务添加到任务看门狗的监控列表中
    esp_task_wdt_add(NULL);

    while (1)
    {
        printf("task2\n");
        //喂狗
        // esp_task_wdt_reset();
        vTaskDelay(1000 /portTICK_PERIOD_MS);
    }
    
}
//------------------------------------------------------------------------------------
void app_main(void)
{
    TaskHandle_t pxTask1 ;
    TaskHandle_t pxTask2 ;
    xTaskCreate(myTask1, "myTask1", 1024, NULL, 1, &pxTask1);
    xTaskCreate(myTask2, "myTask2", 1024, NULL, 1, &pxTask2);

}
```

```C
E (5265) task_wdt: Task watchdog got triggered. The following tasks did not reset the watchdog in time:
E (5265) task_wdt:  - myTask2 (CPU 0)
E (5265) task_wdt: Tasks currently running:
E (5265) task_wdt: CPU 0: IDLE
```

# 十、Queue队列的三种数据传值

![](https://qiniu.yyin.top/20220125142631.png)

Task A 向队列中发送数据，Task B向对列中读取数据。[队列](https://www.cnblogs.com/bigsai/p/11363071.html)就是一个先入先出的数组，一个缓冲区

## 队列创建

```C
QueueHandle_t xQueueCreate( UBaseType_t uxQueueLength,    //队列的长度
                            UBaseType_t uxItemSize );     //队列的宽度
```

## 向队列传递数据

```C
/*向队列发送数据*/
BaseType_t xQueueSend( QueueHandle_t xQueue,           //队列句柄
                       const void * pvItemToQueue,     //传递内容的指针
                       TickType_t xTicksToWait );      //延时
                       
BaseType_t xQueueSendToFront( QueueHandle_t xQueue,
                              const void * pvItemToQueue,
                              TickType_t xTicksToWait );
                              
BaseType_t xQueueSendToBack( QueueHandle_t xQueue,
                             const void * pvItemToQueue,
                             TickType_t xTicksToWait );
```

## 接收数据

```C
BaseType_t xQueueReceive( QueueHandle_t xQueue,  //队列句柄
                          void *pvBuffer,        //一个指向内存中接受数据所预留的空间的指针 
                          TickType_t xTicksToWait );
```

## 示例

### 整型

```C
#include <stdio.h>
#include "sdkconfig.h"
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_system.h"
#include "esp_spi_flash.h"

#include "freertos/queue.h" //新添加的头文件
//------------------------------------------------------------------------------------

void sendTask(void *pvParam)
{   
    //获取参数--队列的句柄
    QueueHandle_t qHandle = (QueueHandle_t)pvParam;
    
    //数据发送状态，如果返回值等于pdPass则表示成功发送数据，否则表示发送失败
    BaseType_t xStatus;
    //数据内容定义
    int i = 0;
    while (1)
    {
        //发送数据
        xStatus = xQueueSend(qHandle,&i, 0);
        //发送数据到队列      
        if(xStatus == pdPASS)
        {
            printf("send data:%d\n",i);
        }
        else
        {
            printf("send data fail\n");
        }
        i++; 
        if(i == 8 ) i= 0;
        vTaskDelay(1000 / portTICK_PERIOD_MS);
    }
}
//------------------------------------------------------------------------------------
void recTask(void *pvParam)
{
    QueueHandle_t qHandle = (QueueHandle_t)pvParam;

    BaseType_t xStatus;

    int j = 0;
    while (1)
    {   
        //超时时间设置为0，表示不等待数据，直接返回
        xStatus = xQueueReceive(qHandle,&j,0);
        if(xStatus == pdPASS)
        {
            printf("rec data:%d\n",j);
        }
        else
        {
            printf("rec data fail\n");
        }

        vTaskDelay(1000 / portTICK_PERIOD_MS);
    }
}
//------------------------------------------------------------------------------------
void app_main(void)
{

    //创建一个句柄，用来接收队列创建函数的指针
    QueueHandle_t qHandle;

    //创建一个长度为5的整形队列
    qHandle = xQueueCreate(5, sizeof(int));

    //如果句柄不等于空，则说明句柄创建成功
    if (qHandle != NULL )
    {
        printf("queue create success\n");
        //将队列的句柄以指针形式传递给任务
        xTaskCreate(sendTask, "sendTask", 1024*5, (void *)qHandle, 0, NULL);
        xTaskCreate(recTask, "recTask", 1024*5, (void *)qHandle, 0, NULL);
    }else{
        printf("queue create failed\n");
    }
}

```

```C
queue create success
send data:0
rec data:0
send data:1
rec data:1
send data:2
rec data:2
send data:3
rec data:3
```

### 结构体

```C
#include <stdio.h>
#include "sdkconfig.h"
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_system.h"
#include "esp_spi_flash.h"

#include "freertos/queue.h" //新添加的头文件

typedef struct A_STRUCT
{
    char id;
    char data;
} xStruct;
//------------------------------------------------------------------------------------

void sendTask(void *pvParam)
{   
    //获取参数--队列的句柄
    QueueHandle_t qHandle = (QueueHandle_t)pvParam;
    
    //数据发送状态，如果返回值等于pdPass则表示成功发送数据，否则表示发送失败
    BaseType_t xStatus;

    //结构体
    xStruct xUSB  = {1,55};

    while (1)
    {
        //发送数据
        xStatus = xQueueSend(qHandle,&xUSB, 0);
        //发送数据到队列      
        if(xStatus == pdPASS)
        {
            printf("send id:%d data:%d\n",xUSB.id,xUSB.data);
        }
        else
        {
            printf("send data fail\n");
        }

        vTaskDelay(1000 / portTICK_PERIOD_MS);
    }
}
//------------------------------------------------------------------------------------
void recTask(void *pvParam)
{
    QueueHandle_t qHandle = (QueueHandle_t)pvParam;

    BaseType_t xStatus;

    //结构体
    xStruct xUSB  = {0,0};
    while (1)
    {   
        //超时时间设置为0，表示不等待数据，直接返回
        xStatus = xQueueReceive(qHandle,&xUSB,0);
        if(xStatus == pdPASS)
        {
            printf("rec id:%d data:%d\n",xUSB.id,xUSB.data);
        }
        else
        {
            printf("rec data fail\n");
        }

        vTaskDelay(1000 / portTICK_PERIOD_MS);
    }
}
//------------------------------------------------------------------------------------
void app_main(void)
{

    //创建一个句柄，用来接收队列创建函数的指针
    QueueHandle_t qHandle;

    //创建一个长度为5的整形队列
    qHandle = xQueueCreate(5, sizeof(xStruct));

    //如果句柄不等于空，则说明句柄创建成功
    if (qHandle != NULL )
    {
        printf("queue create success\n");
        //将队列的句柄以指针形式传递给任务
        xTaskCreate(sendTask, "sendTask", 1024*5, (void *)qHandle, 0, NULL);
        xTaskCreate(recTask, "recTask", 1024*5, (void *)qHandle, 0, NULL);
    }else{
        printf("queue create failed\n");
    }
}

```

### 指针

```C
#include <stdio.h>
#include "sdkconfig.h"
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_system.h"
#include "esp_spi_flash.h"

#include "freertos/queue.h" //新添加的头文件


//------------------------------------------------------------------------------------

void sendTask(void *pvParam)
{   
    //获取参数--队列的句柄
    QueueHandle_t qHandle = (QueueHandle_t)pvParam;
    
    //数据发送状态，如果返回值等于pdPass则表示成功发送数据，否则表示发送失败
    BaseType_t xStatus;

    //指针
    char *pData;

    while (1)
    {
        //分配内存
        pData = (char *)malloc(sizeof(char) * 50);
        //向指针写入数据
        snprintf(pData, 50, "Hello World! %d \n", xTaskGetTickCount());
        //发送数据
        xStatus = xQueueSend(qHandle,&pData, 0);
        //发送数据到队列      
        if(xStatus == pdPASS)
        {
            printf("send deone\n");
        }
        else
        {
            printf("send data fail\n");
        }
        //释放内存
        free(pData);
        vTaskDelay(1000 / portTICK_PERIOD_MS);
    }
}
//------------------------------------------------------------------------------------
void recTask(void *pvParam)
{
    QueueHandle_t qHandle = (QueueHandle_t)pvParam;

    BaseType_t xStatus;

    //指针
    char *pData;

    while (1)
    {  
        if(uxQueueMessagesWaiting(QHandle)!=0)
        {
            xStatus = xQueueReceive(qHandle，&i，0);
            if(xStatus != pdPASS) printf("rec fail\n");
            else printf("rec: %s!\n",pData);
        }else{
            printf("no data!\n");
        }
        
        vTaskDelay(1000 / portTICK_PERIOD_MS);
    }
}
//------------------------------------------------------------------------------------
void app_main(void)
{

    //创建一个句柄，用来接收队列创建函数的指针
    QueueHandle_t qHandle;

    //创建一个长度为5的整形队列
    qHandle = xQueueCreate(5, sizeof(char *));

    //如果句柄不等于空，则说明句柄创建成功
    if (qHandle != NULL )
    {
        printf("queue create success\n");
        //将队列的句柄以指针形式传递给任务
        xTaskCreate(sendTask, "sendTask", 1024*5, (void *)qHandle, 0, NULL);
        xTaskCreate(recTask, "recTask", 1024*5, (void *)qHandle, 0, NULL);
    }else{
        printf("queue create failed\n");
    }
}

```

注意：对指针的使用一定要注意释放 内存，否则可能stack overflow

# 十一、Queue队列多进单出

![](https://qiniu.yyin.top/20220125170131.png)

对于这样的模型，优先级别的设置十分重要

```C
#include <stdio.h>
#include "sdkconfig.h"
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_system.h"
#include "esp_spi_flash.h"

#include "freertos/queue.h" //新添加的头文件


//------------------------------------------------------------------------------------

void sendTask1(void *pvParam)
{   
    //获取参数--队列的句柄
    QueueHandle_t qHandle = (QueueHandle_t)pvParam;
    
    //数据发送状态，如果返回值等于pdPass则表示成功发送数据，否则表示发送失败
    BaseType_t xStatus;

    int i=111;

    while (1)
    {
        //发送数据
        xStatus = xQueueSend(qHandle,&i, 0);
        //发送数据到队列      
        if(xStatus == pdPASS)
        {
            printf("task 1 send deone\n");
        }
        else
        {
            printf("task 1 send data fail\n");
        }

        vTaskDelay(1000 / portTICK_PERIOD_MS);
    }
}
//------------------------------------------------------------------------------------

void sendTask2(void *pvParam)
{   
    //获取参数--队列的句柄
    QueueHandle_t qHandle = (QueueHandle_t)pvParam;
    
    //数据发送状态，如果返回值等于pdPass则表示成功发送数据，否则表示发送失败
    BaseType_t xStatus;

    int i=222;

    while (1)
    {
        //发送数据
        xStatus = xQueueSend(qHandle,&i, 0);
        //发送数据到队列      
        if(xStatus == pdPASS)
        {
            printf("task 2 send deone\n");
        }
        else
        {
            printf("task 2 send data fail\n");
        }

        vTaskDelay(1000 / portTICK_PERIOD_MS);
    }
}
//------------------------------------------------------------------------------------
void recTask(void *pvParam)
{
    QueueHandle_t qHandle = (QueueHandle_t)pvParam;

    BaseType_t xStatus;

    int i;

    while (1)
    {  //判断队列是否为空
        if(uxQueueMessagesWaiting(qHandle)!=0)
        {   
            //利用portMAX_DELAY参数，将recTask阻塞在这里，直到队列中有数据
            //又因为recTask是优先级别比较高的任务，所以只要队列中有数据，recTask就会立即被唤醒
            //而sendTask1和sendTask2的优先级是相同的,所以二者会轮流发送数据
            xStatus = xQueueReceive(qHandle,&i,portMAX_DELAY);
            if(xStatus != pdPASS) printf("rec fail\n");
            else printf("rec: %d!\n",i);
        }else{
            printf("no data!\n");
        }
        
        vTaskDelay(1000 / portTICK_PERIOD_MS);
    }
}
//------------------------------------------------------------------------------------
void app_main(void)
{

    //创建一个句柄，用来接收队列创建函数的指针
    QueueHandle_t qHandle;

    //创建一个长度为5的整形队列
    qHandle = xQueueCreate(5, sizeof(int));

    //如果句柄不等于空，则说明句柄创建成功
    if (qHandle != NULL )
    {
        printf("queue create success\n");
        //将队列的句柄以指针形式传递给任务
        //我们需要根据实际的需求,设置任务的优先级
        xTaskCreate(sendTask1, "sendTask1", 1024*5, (void *)qHandle, 1, NULL);
        xTaskCreate(sendTask2, "sendTask2", 1024*5, (void *)qHandle, 1, NULL);
        xTaskCreate(recTask, "recTask", 1024*5, (void *)qHandle, 2, NULL);
    }else{
        printf("queue create failed\n");
    }
}

```

# 十二、Queue Set队列集合

**面对这样一个情况**：

多个Task发送数据给各自的Queue，在由一个Controller读出数据

![](https://qiniu.yyin.top/20220130221635.png)

针对这样的数据模型，FreeRTOS给出了一个队列集合的概念，即Queue Set

把以上的多个队列在一起，FreeRTOS提供一个函数接口，让读数据的Controller从集合中取得有数据的对应的**队列的句柄**，然后再从对应的队列中读出数据。

## 队列集合的创建

```C
QueueSetHandle_t xQueueCreateSet( const UBaseType_t uxEventQueueLength );  //集合的长度
```

## 将队列添加到集合中

```C
BaseType_t xQueueAddToSet( QueueSetMemberHandle_t xQueueOrSemaphore,  //队列的句柄
                           QueueSetHandle_t xQueueSet );  //集合的句柄
```

## 选择一个有数据的队列出来

```C
QueueSetMemberHandle_t xQueueSelectFromSet( QueueSetHandle_t xQueueSet,
                                            const TickType_t xTicksToWait );
```

```C
BaseType_t xQueueReceive( QueueHandle_t xQueue,
                          void *pvBuffer,TickType_t xTicksToWait );

```

## 示例

```C
#include <stdio.h>
#include "sdkconfig.h"
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_system.h"
#include "esp_spi_flash.h"

#include "freertos/queue.h" //新添加的头文件

//------------------------------------------------------------------------------------

void sendTask1(void *pvParam)
{
    QueueHandle_t qHandle = (QueueHandle_t)pvParam;

    BaseType_t xStatus;

    int i = 111;

    while (1)
    {
        //发送数据
        xStatus = xQueueSend(qHandle, &i, 0);

        if (xStatus == pdPASS)
        {
            printf("task 1 send deone\n");
        }
        else
        {
            printf("task 1 send data fail\n");
        }

        vTaskDelay(1000 / portTICK_PERIOD_MS);
    }
}
//------------------------------------------------------------------------------------

void sendTask2(void *pvParam)
{
    QueueHandle_t qHandle = (QueueHandle_t)pvParam;

    BaseType_t xStatus;

    int i = 222;

    while (1)
    {
        xStatus = xQueueSend(qHandle, &i, 0);

        if (xStatus == pdPASS)
        {
            printf("task 2 send deone\n");
        }
        else
        {
            printf("task 2 send data fail\n");
        }

        vTaskDelay(1000 / portTICK_PERIOD_MS);
    }
}
//------------------------------------------------------------------------------------
//在recvTask中获取队列集合的
void recTask(void *pvParam)
{
    QueueSetHandle_t queueSet = (QueueSetHandle_t)pvParam;

    BaseType_t xStatus;

    int i;

    //表示取得的有数据的Queue
    QueueSetMemberHandle_t queueData;

    while (1)
    {
        queueData = xQueueSelectFromSet(queueSet, portMAX_DELAY);
        xStatus = xQueueReceive(queueData, &i, portMAX_DELAY);

        if (xStatus != pdPASS)
            printf("rec fail\n");
        else
            printf("rec: %d!\n", i);
    }
}
//------------------------------------------------------------------------------------
void app_main(void)
{

    QueueHandle_t qHandle1;
    qHandle1 = xQueueCreate(5, sizeof(int));

    QueueHandle_t qHandle2;
    qHandle2 = xQueueCreate(5, sizeof(int));

    //创建队列集合，参数为Queue的长度，即5+5=10
    QueueSetHandle_t qSetHandle;
    qSetHandle = xQueueCreateSet(10);

    //将队列1和队列2加入到队列集合中,然后将队列集合的句柄传递给接收任务
    xQueueAddToSet(qHandle1, qSetHandle);
    xQueueAddToSet(qHandle2, qSetHandle);

    //如果句柄不等于空，则说明句柄创建成功
    if ((qHandle1 != NULL) && (qHandle2 != NULL) && (qSetHandle != NULL))
    {
        printf("queue create success\n");
        xTaskCreate(sendTask1, "sendTask1", 1024 * 5, (void *)qHandle1, 1, NULL);
        xTaskCreate(sendTask2, "sendTask2", 1024 * 5, (void *)qHandle2, 1, NULL);
        // 将队列集合传递给recTask
        xTaskCreate(recTask, "recTask", 1024 * 5, (void *)qSetHandle, 2, NULL);
    }
    else
    {
        printf("queue create failed\n");
    }
}


```

# 十三、Queue MailBox 队列邮箱

![](https://qiniu.yyin.top/mailbox.png)

在FreeRTOS当中`mailbox is used to refer to a  queue that has a length of one`,这个队列只包含有一个数据。

普通的队列只是由一个task传递到另一个task或者中断,发送方将一个项目放入队列中，而接收方从队列中移除该项。 数据需要从发送方经过队列的接收机。

队列邮箱用来保存数据，以供任务任务或任何中断来读取的数据。数据不会传递出邮箱，而是留在邮箱中，直到被覆盖。其他的task只能读取这个数据，发件方将覆盖邮箱中的值。接收方从邮箱读取该值，但不从邮箱删除该值。

## 向mailbox写入数据

```C
BaseType_t xQueueOverwrite( QueueHandle_t xQueue,          //mailbox的句柄
                            const void *pvItemToQueue );   //写入的值

```

## 从mailbox读取数据

```C
BaseType_t xQueuePeek( QueueHandle_t xQueue,     //mailbox的句柄
                       void *pvBuffer,           //目标指针
                       TickType_txTicksToWait ); //阻塞时间
```

## 示例

```C
#include <stdio.h>
#include "sdkconfig.h"
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_system.h"
#include "esp_spi_flash.h"

#include "freertos/queue.h" //新添加的头文件

//------------------------------------------------------------------------------------
void writeTask(void *pvParam)
{
    QueueHandle_t mailbox = (QueueHandle_t)pvParam;
    BaseType_t xStatus;
    int i = 0;
    while (1)
    {
        xStatus = xQueueOverwrite(mailbox,&i);
        if (xStatus == pdPASS)
            printf("writeTask send deone\n");
        else
            printf("writeTask send data fail\n");
        i++;
        vTaskDelay(1000 / portTICK_PERIOD_MS);
    }
}
//------------------------------------------------------------------------------------
void readTask(void *pvParam)
{
    QueueHandle_t mailbox = (QueueHandle_t)pvParam;
    BaseType_t xStatus;
    int i;

    while (1)
    {
        xStatus = xQueuePeek(mailbox,&i,0);
        if (xStatus != pdPASS)
            printf("read fail\n");
        else
            printf("read %s: %d!\n",pcTaskGetTaskName(NULL), i);
        //延时1秒
        vTaskDelay(1000 / portTICK_PERIOD_MS);
    }
}
//------------------------------------------------------------------------------------
void app_main(void)
{


    QueueHandle_t mailbox = xQueueCreate(1, sizeof(int));

    //如果句柄不等于空，则说明句柄创建成功
    if ((mailbox != NULL))
    {
        printf("queue create success\n");

        xTaskCreate(writeTask, "writeTask", 1024 * 5, (void *)mailbox, 1, NULL);
        //共用同一个函数体
        xTaskCreate(readTask, "sendTask1", 1024 * 5, (void *)mailbox, 2, NULL);
        xTaskCreate(readTask, "sendTask2", 1024 * 5, (void *)mailbox, 2, NULL);
        xTaskCreate(readTask, "sendTask3", 1024 * 5, (void *)mailbox, 2, NULL);
        
    }
    else
    {
        printf("queue create failed\n");
    }
}

```

# 十四、Software Timer软件定时器

软件定时器是用程序模拟出来的定时器，可以由一个硬件定时器模拟出成千上万个软件定时器，这样程序在需要使用较多定时器的时候就不会受限于硬件资源的不足，这是软件定时器的一个优点，即数量不受限制。但由于软件定时器是通过程序实现的，其运行和维护都需要耗费一定的CPU资源，同时精度也相对硬件定时器要差一些。

在Linux，uC/OS，[FreeRTOS](https://so.csdn.net/so/search?q=FreeRTOS&spm=1001.2101.3001.7020)等操作系统中，都带有软件定时器，原理大同小异。典型的实现方法是：通过一个硬件定时器产生固定的时钟节拍，每次硬件定时器中断到，就对一个全局的时间标记加一，每个软件定时器都保存着到期时间，程序需要定期扫描所有运行中的软件定时器，将各个到期时间与全局时钟标记做比较，以判断对应软件定时器是否到期，到期则执行相应的回调函数，并关闭该定时器。

以上是单次定时器的实现，若要实现周期定时器，即到期后接着重新定时，只需要在执行完[回调函数](https://www.wolai.com/7SLgWPWAhSBRh8pMRBXN8k)后，获取当前时间标记的值，加上延时时间作为下一次到期时间，继续运行软件定时器即可。

**软件定时器与硬件定时器的区别**

1. 软件定时器与平台无关，与硬件无关
2. 可以定义多个软件定时器

## 创建软件定时器

```C
#include “FreeRTOS.h”
#include “timers.h”
/*用于创建软件定时器*/
TimerHandle_t xTimerCreate( const char *pcTimerName,          //定时器的名称
                            const TickType_t xTimerPeriod,    //定时器的周期
                            const UBaseType_t uxAutoReload,   //设定定时器的重载属性，如果反复执行设定为pdTrue
                            void * const pvTimerID,           //定时器的id
                            TimerCallbackFunction_t pxCallbackFunction ); //定时器的回调函数
```

## 启动软件定时器

```C
BaseType_t xTimerStart( TimerHandle_t xTimer,       //软件定时器创建的句柄
                        TickType_t xTicksToWait );  //当守护进程不能接受对应的内容,我们设置等待
```

## 停止定时器运行

```C
BaseType_t xTimerStop( TimerHandle_t xTimer,       //定时器句柄
                       TickType_t xTicksToWait );  //延时
```

## 获取软件定时器的名字

```C
const char * pcTimerGetName( TimerHandle_t xTimer ); 
```

## 获取软件定时器的id

```C
void *pvTimerGetTimerID( TimerHandle_t xTimer );
```

## 重启软件定时器

```C
BaseType_t xTimerReset( TimerHandle_t xTimer, TickType_t xTicksToWait );
```

## 更改软件定时器的周期

```C
BaseType_t xTimerChangePeriod( TimerHandle_t xTimer,
                               TickType_t xNewPeriod,    //重新定义定时器的运行周期
                               TickType_t xTicksToWait );
```

可以用于控制led的闪烁频率

## 示例

```C
#include <stdio.h>
#include "sdkconfig.h"
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_system.h"
#include "esp_spi_flash.h"

#include "freertos/timers.h" //新添加的头文件

//------------------------------------------------------------------------------------
void TimerCallback(TimerHandle_t xTimer)
{
    printf("Timer name = %s \n", (char *)pcTimerGetTimerName(xTimer));

    uint32_t id;
    id = (uint32_t)pvTimerGetTimerID(xTimer);
    printf("Timer ID = %d \n", id);

}
//------------------------------------------------------------------------------------
void app_main(void)
{
    TimerHandle_t xTimer1;
    TimerHandle_t xTimer2;
    // TiCKS是CUP运行的次数，每次CUP运行的时间是1ms,通过宏定义转换
    //  xTimer = xTimerCreate("Timer", pdMS_TO_TICKS(1000), pdFALSE, NULL, Timer1Callback);

    //周期性运行
    xTimer1 = xTimerCreate("Timer1", pdMS_TO_TICKS(1000), pdTRUE, (void *)0, TimerCallback);
    xTimer2 = xTimerCreate("Timer2", pdMS_TO_TICKS(2000), pdTRUE, (void *)1, TimerCallback);
    //启动软件定时器
    xTimerStart(xTimer1, 0);
    xTimerStart(xTimer2, 0);

    // while(1){
    //     vTaskDelay(pdMS_TO_TICKS(1000));
    //     xTimerReset(xTimer2, 0);
    // }

    // vTaskDelay(pdMS_TO_TICKS(6000));
    // xTimerStop(xTimer,0);

    xTimerChangePeriod(xTimer1, pdMS_TO_TICKS(2000), 0);

}

```

# 十五、Binary Semaphore 二进制信号量

信号量可以我们的task进行同步;用于让我们控制那些任务现在执行,那些任务需要暂停.

二进制信号量,只有两个值,在1的时候开始执行,在0的时候停止执行.

## 创建二进制信号量

```C
#include “FreeRTOS.h”
#include “semphr.h”
SemaphoreHandle_t xSemaphoreCreateBinary( void );
```

## 释放信号量

```C
BaseType_t xSemaphoreGive( SemaphoreHandle_t xSemaphore );
```

## 获取信号量

```C
BaseType_t xSemaphoreTake( SemaphoreHandle_t xSemaphore, TickType_t xTicksToWait );
```

## 示例

```C
#include <stdio.h>
#include "sdkconfig.h"
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_system.h"
#include "esp_spi_flash.h"

#include "freertos/semphr.h" //新添加的头文件

//------------------------------------------------------------------------------------
//假设有两个任务 ,同时操控iCount的值
int iCount = 0;
SemaphoreHandle_t xSemaphore;

//------------------------------------------------------------------------------------
void myTask1(void * pvParam)
{
    while (1)
    {
        //获取信号量,如果没有获取,就一直等到
        xSemaphoreTake(xSemaphore, portMAX_DELAY);

        for (int i = 0; i < 10; i++)
        {
            iCount++;
            printf("Task1: iCount = %d\n", iCount);
            vTaskDelay(1000 / portTICK_PERIOD_MS);
        }
        //释放信号量
        xSemaphoreGive(xSemaphore);
        vTaskDelay(1000 / portTICK_PERIOD_MS);
    }
    
}
//------------------------------------------------------------------------------------
void myTask2(void * pvParam)
{
    while (1)
    {
        //获取信号量,如果没有获取,就一直等到
        xSemaphoreTake(xSemaphore, portMAX_DELAY);

        for (int i = 0; i < 10; i++)
        {
            iCount++;
            printf("Task2: iCount = %d\n", iCount);
            vTaskDelay(1000 / portTICK_PERIOD_MS);
        }
        //释放信号量
        xSemaphoreGive(xSemaphore);
        vTaskDelay(1000 / portTICK_PERIOD_MS);
    }
    
}
//------------------------------------------------------------------------------------
void app_main(void)
{
    //创建信号量
    xSemaphore = xSemaphoreCreateBinary();
    //释放信号量
    xSemaphoreGive(xSemaphore);
    //创建任务
    xTaskCreate(myTask1, "myTask1", 1024*5, NULL, 1, NULL);
    xTaskCreate(myTask2, "myTask2", 1024*5, NULL, 1, NULL);
}

```

# 十六、Count Semaphore 计数型信号量

好比一个停车场,里面有10个车位,当有车进入的时候计数器减1,当车开出来的时候,车位清空,计数器加1

当计数器为0时表示没有可用的空间.

## 创建计数型信号量

```C
SemaphoreHandle_t xSemaphoreCreateCounting( UBaseType_t uxMaxCount,        //信号量的最大数
                                            UBaseType_t uxInitialCount );  //信号量的chusdhi数值
```

## 获取计数型信号量的数目

```C
UBaseType_t uxSemaphoreGetCount( SemaphoreHandle_t xSemaphore );
```

## 示例

```C
#include <stdio.h>
#include "sdkconfig.h"
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_system.h"
#include "esp_spi_flash.h"

#include "freertos/semphr.h" //新添加的头文件

//------------------------------------------------------------------------------------

SemaphoreHandle_t xSemaphore;

//------------------------------------------------------------------------------------
void carInTask(void * pvParam)
{
    int emptySpace = 0;
    BaseType_t iResult;
    while (1)
    {
        emptySpace = uxSemaphoreGetCount(xSemaphore);
        printf(" emptySpace = %d\n",emptySpace);
        
        //获取一个信号量的资源
        iResult =xSemaphoreTake(xSemaphore, 0);
        
        //检查是否成功获取信号量
        if(iResult == pdPASS)
            printf("One car in!\n");
        else
            printf("No space!\n");
        vTaskDelay(1000 / portTICK_PERIOD_MS);
    }
    
}
//------------------------------------------------------------------------------------
void carOutTask(void * pvParam)
{
    while (1)
    {
        vTaskDelay(6000 / portTICK_PERIOD_MS);
        //释放信号量
        xSemaphoreGive(xSemaphore);
        printf("One car out!\n");
    }
    
    
}
//------------------------------------------------------------------------------------
void app_main(void)
{
    //创建计数型信号量
    xSemaphore = xSemaphoreCreateCounting(5,5);
    //释放信号量
    xSemaphoreGive(xSemaphore);
    //创建任务
    xTaskCreate(carInTask, "carInTask", 1024*5, NULL, 1, NULL);
    xTaskCreate(carOutTask, "carOutTask", 1024*5, NULL, 1, NULL);
}

```

# 十七、Mutex 互斥量

互相排斥的信号量,与二进制信号量的区别:互斥量会继承优先级别

拥有互斥量的任务,会继承,企图获得同样互斥量任务的优先级别,"attempting to 'take' the same mutex.

## 创建互斥变量

```C
SemaphoreHandle_t xSemaphoreCreateMutex( void );
```

## 示例

app_main

→ task3,存在阻塞

→ task2,存在阻塞

→ task1,获取互斥量  

→ task1任务超时,但task1此时拥有互斥量;而此时task3 执行,企图获取互斥量,所以task1在拥有互斥量的前提下,继承了task3的优先级,继续执行.

→ task1结束for循环,释放互斥量,恢复之前的优先级;task3成功获取互斥量,while(1)循环反复执行;task2长时间阻塞,触发系统看门狗.

```C
#include <stdio.h>
#include "sdkconfig.h"
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_system.h"
#include "esp_spi_flash.h"

#include "freertos/semphr.h" //新添加的头文件
//------------------------------------------------------------------------------------
SemaphoreHandle_t mutexHandle;
//------------------------------------------------------------------------------------
void Task1(void *pvParameters)
{
    BaseType_t iResult;
    while (1)
    {
        printf("task1 begin\n");

        vTaskDelay(pdMS_TO_TICKS(1000));
        //取得互斥量的所有权
        iResult = xSemaphoreTake(mutexHandle, 1000);
        if (iResult == pdPASS)
        {
            printf("task1 get mutex\n");
            for (int i = 0; i < 50; i++)
            {
                //超时
                printf("task1 i:%d\n",i);
                vTaskDelay(pdMS_TO_TICKS(1000));
            }
            xSemaphoreGive(mutexHandle);
            printf("task1 give mutex!\n");

            vTaskDelay(pdMS_TO_TICKS(5000));
        }
        else
        {
            printf("task1 get mutex failed!\n");
            vTaskDelay(pdMS_TO_TICKS(1000));
        }
        
        
    }
    
}
//------------------------------------------------------------------------------------
void Task2(void *pvParameters)
{
    printf("task2 begin\n");
    //阻塞task2,让task1有机会执行
    vTaskDelay(pdMS_TO_TICKS(10000));

    while(1)
    {
        ;
    }

}
//------------------------------------------------------------------------------------
void Task3(void *pvParameters)
{
    BaseType_t iResult;

    printf("task3 begin\n");
    //阻塞自身
    vTaskDelay(pdMS_TO_TICKS(10000));

    while(1){
        //试图取得信号量
        iResult = xSemaphoreTake(mutexHandle,1000);
        if(iResult == pdPASS){
            printf("task3 get mutex\n");
            for (int i = 0; i < 10; i++)
            {
                printf("task3 i:%d\n",i);
                vTaskDelay(pdMS_TO_TICKS(1000));
            }
            //释放信号量
            xSemaphoreGive(mutexHandle);
            printf("task3 give mutex\n");
            vTaskDelay(pdMS_TO_TICKS(5000));
        }
        else{
            printf("task3 get mutex fail\n");
            vTaskDelay(pdMS_TO_TICKS(1000));
        }
    }
}
//------------------------------------------------------------------------------------
void app_main(void)
{
    mutexHandle = xSemaphoreCreateMutex();
    //挂起任务调度器
    vTaskSuspendAll();
   //创建任务
    xTaskCreate(Task1, "Task1", 1024*5, NULL, 1, NULL);
    xTaskCreate(Task2, "Task2", 1024*5, NULL, 2, NULL);
    xTaskCreate(Task3, "Task3", 1024*5, NULL, 3, NULL);
    //恢复任务调度器
    xTaskResumeAll();
}

```

# 十八、Recursive Mutex递归互斥

假设我们有1个task和两个资源A和B.Task首先获得资源A。在资源A获得一定运算处理后,还需要获得资源B进行运算处理。处理完资源B后回到资源A,并释放资源B,然后处理完资源A后回到Task.

![](https://qiniu.yyin.top/recursiveMutex.png)

一般来说,想要实现这样的任务,我们需要多把互斥锁,为了简化操作,我们引入递归互斥量。递归互斥量允许我们只使用一个变量,在每次使用资源的时候上一把锁,释放一个资源就释放一把锁。

## 创建递归互斥信号量

```C
SemaphoreHandle_t xSemaphoreCreateRecursiveMutex( void );
```

## 示例

```C
#include <stdio.h>
#include "sdkconfig.h"
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_system.h"
#include "esp_spi_flash.h"

#include "freertos/semphr.h" //新添加的头文件

//------------------------------------------------------------------------------------
SemaphoreHandle_t mutexHandle;
//------------------------------------------------------------------------------------
void Task1(void *pvParameters)
{
    while (1)
    {
        printf("------------\n");
        printf("task1 begin!\n");

        // A添加递归锁
        xSemaphoreTakeRecursive(mutexHandle, portMAX_DELAY);
        printf("task1 take A!\n");

        //模拟运算
        for (int i = 0; i < 10; i++)
        {
            printf("task1 i=%d for A!\n", i);
            vTaskDelay(pdMS_TO_TICKS(1000));
        }

        // B添加递归锁
        xSemaphoreTakeRecursive(mutexHandle, portMAX_DELAY);
        printf("task1 take B!\n");

        //模拟运算
        for (int i = 0; i < 10; i++)
        {
            printf("task1 i=%d for B!\n", i);
            vTaskDelay(pdMS_TO_TICKS(1000));
        }
        printf("task1 give B!\n");
        xSemaphoreGiveRecursive(mutexHandle);

        vTaskDelay(pdMS_TO_TICKS(3000));

        printf("task1 give A!\n");
        xSemaphoreGiveRecursive(mutexHandle);

        vTaskDelay(pdMS_TO_TICKS(3000));
    }
}
//------------------------------------------------------------------------------------
void Task2(void *pvParameters)
{

    vTaskDelay(pdMS_TO_TICKS(1000));

    while (1)
    {
        printf("------------\n");
        printf("task2 begin!\n");

        //添加递归锁
        xSemaphoreTakeRecursive(mutexHandle, portMAX_DELAY);
        printf("task2 take !\n");

        //模拟运算
        for (int i = 0; i < 10; i++)
        {
            printf("task2 i=%d for A!\n", i);
            vTaskDelay(pdMS_TO_TICKS(1000));
        }
        printf("task2 give A!\n");
        xSemaphoreGiveRecursive(mutexHandle);

        vTaskDelay(pdMS_TO_TICKS(3000));
    }
}
//------------------------------------------------------------------------------------

//------------------------------------------------------------------------------------
void app_main(void)
{

    mutexHandle = xSemaphoreCreateRecursiveMutex();
    //挂起任务调度器
    vTaskSuspendAll();
    //创建任务
    xTaskCreate(Task1, "Task1", 1024 * 5, NULL, 1, NULL);
    xTaskCreate(Task2, "Task2", 1024 * 5, NULL, 1, NULL);
    //恢复任务调度器
    xTaskResumeAll();
}

```

# 十九、Event Group Wait事件组等待

用于协调各种任务之间的运行顺序,

## 创建事件组

```C
#include “FreeRTOS.h”
#include “event_groups.h”
EventGroupHandle_t xEventGroupCreate( void );
```

注意检查`configUSE_16_BIT_TICKS`的宏定义的参数;1代表8位,0代表24位

## 事件组等待

这个函数是一个task用来检查当前的事件组相应的位是否设置或者清除,若被设置则继续运行,如果没有被设置则可以选择进入阻塞状态,且不能在中断中调用.等待事件组对应位数进行触发.

```C
EventBits_t xEventGroupWaitBits( const EventGroupHandle_t xEventGroup,   //事件组的句柄
                                 const EventBits_t uxBitsToWaitFor,      //显示要检查那些位如0x05-->101 是检查0和2位
                                 const BaseType_t xClearOnExit,          //在退出时是否清零
                                 const BaseType_t xWaitForAllBits,       //要求对应位为1是同时满足,或者任意满足
                                 TickType_t xTicksToWait );              //延迟
```

## 清除事件组

```C
void vEventGroupDelete( EventGroupHandle_t xEventGroup );
```

## 取得事件组的位数

```C
EventBits_t xEventGroupGetBits( EventGroupHandle_t xEventGroup );
```

## 设置事件组的位数

```C
EventBits_t xEventGroupSetBits( EventGroupHandle_t xEventGroup,   //事件组的句柄
                                const EventBits_t uxBitsToSet );  //设置的位
```

## 示例

```C
#include <stdio.h>
#include "sdkconfig.h"
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_system.h"
#include "esp_spi_flash.h"

#include "freertos/event_groups.h"

//------------------------------------------------------------------------------------
//定义EventGroup的句柄
EventGroupHandle_t xEventGroup;
//------------------------------------------------------------------------------------
void Task1(void *pvParameters)
{
    while (1)
    {
        printf("------------\n");
        printf("task1 begin!\n");

        //创建事件组等待,使得当前任务进行阻塞等待
        xEventGroupWaitBits(xEventGroup,    //事件组的句柄
                            BIT0 | BIT4,  //显示要检查那些位如0x05-->101 是检查0和2位
                            pdTRUE,         //在退出时是否清零
                            pdFALSE,        //要求对应位为1是同时满足pdTRUE,或者任意满足pdFASLE,将事件组唤醒
                            portMAX_DELAY); //等待的时间

        printf("------------\n");
        printf("BIT0 OR BIT4 is set!\n");
        vTaskDelay(pdMS_TO_TICKS(1000));
    }
}
//------------------------------------------------------------------------------------
void Task2(void *pvParameters)
{
    vTaskDelay(pdMS_TO_TICKS(1000));
    while (1)
    {
        printf("------------\n");
        printf("Task2 begin to set BIT_0!\n");
        vTaskDelay(pdMS_TO_TICKS(1000));
        xEventGroupSetBits(xEventGroup, BIT0); 
        
        printf("------------\n");
        printf("Task2 begin to set BIT_4!\n");
        vTaskDelay(pdMS_TO_TICKS(1000));
        xEventGroupSetBits(xEventGroup, BIT4);

        vTaskDelay(pdMS_TO_TICKS(1000));
    }
}
//------------------------------------------------------------------------------------

//------------------------------------------------------------------------------------
void app_main(void)
{

    //创建事件组
    xEventGroup = xEventGroupCreate();

    if (xEventGroup == NULL)
    {
        printf("xEventGroup Create failed!\n");
    }
    else
    {
        printf("xEventGroup Create success!\n");
        //挂起所有任务
        vTaskSuspendAll();

        xTaskCreate(Task1, "Task1", 1024 * 5, NULL, 1, NULL);
        xTaskCreate(Task2, "Task2", 1024 * 5, NULL, 1, NULL);

        xTaskResumeAll();
    }
}

```

# 二十、Event Group Sync事件组的同步

假设我们有三个任务task1、task2、task3,对于事件组等待

```Mermaid
flowchart LR
  id1(task1) --运行一段时间-->  id4((wait))
  id4 --> id9[继续执行]
  id2(task2) --> id5((set))-.设置位.-> id4 
  id5 --> id7[继续执行]
  id3(task3) -->id6((set)) -.设置位.-> id4
  id6 --> id8[继续执行]

```

对于事件组同步

```Mermaid
flowchart LR
  id1(task1) -->  id4((sync))
  id4 --> id9[事件位全部被设置]
  id2(task2) --> id5((sync)) --设置事件位--> id7[阻塞自身]
  id7 -..-> id9
  id3(task3) -->id6((sync)) --设置事件位--> id8[阻塞自身]
  id6 -..-> id9
  id9 ==> id10[三个 task 同时运行]
```

## 创建同步事件组

```C
EventBits_t xEventGroupSync( EventGroupHandle_t xEventGroup,      //事件组的句柄
                             const EventBits_t uxBitsToSet,       //设置一个bit
                             const EventBits_t uxBitsToWaitFor,   //等待其他task设置对应地bit
                             TickType_t xTicksToWait );
```

## 示例

```C
#include <stdio.h>
#include "sdkconfig.h"
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_system.h"
#include "esp_spi_flash.h"

#include "freertos/event_groups.h"

//------------------------------------------------------------------------------------
//定义EventGroup的句柄
EventGroupHandle_t xEventGroup;

#define TASK_0_BIT ( 1 << 0 )
#define TASK_1_BIT ( 1 << 1 )
#define TASK_2_BIT ( 1 << 2 )
#define ALL_SYNC_BITS ( TASK_0_BIT | TASK_1_BIT | TASK_2_BIT )
//------------------------------------------------------------------------------------
void Task0(void *pvParameters)
{
    while(1)
    {
        printf("------------------Task0------------------\n");
        printf("Task0 begin!\n");

        vTaskDelay(pdMS_TO_TICKS(1000));

        printf("Task0 set TASK_0_BIT!\n");
        
        //创建同步事件组
        xEventGroupSync(xEventGroup,
                        TASK_0_BIT,
                        ALL_SYNC_BITS,
                        portMAX_DELAY);
        
        printf("Task0 sync!\n");
        vTaskDelay(pdMS_TO_TICKS(10000));
    }
}
//------------------------------------------------------------------------------------
void Task1(void *pvParameters)
{
    while(1)
    {
        printf("------------------Task1------------------\n");
        printf("Task1 begin!\n");

        vTaskDelay(pdMS_TO_TICKS(3000));

        printf("Task1 set TASK_1_BIT!\n");
        
        //创建同步事件组
        xEventGroupSync(xEventGroup,
                        TASK_1_BIT,
                        ALL_SYNC_BITS,
                        portMAX_DELAY);
        
        printf("Task1 sync!\n");
        vTaskDelay(pdMS_TO_TICKS(10000));
    }
}
//------------------------------------------------------------------------------------
void Task2(void *pvParameters)
{
    while(1)
    {
        printf("------------------Task2------------------\n");
        printf("Task2 begin!\n");

        vTaskDelay(pdMS_TO_TICKS(5000));

        printf("Task2 set TASK_2_BIT!\n");
        
        //创建同步事件组
        xEventGroupSync(xEventGroup,
                        TASK_2_BIT,
                        ALL_SYNC_BITS,
                        portMAX_DELAY);
        
        printf("Task2 sync!\n");
        vTaskDelay(pdMS_TO_TICKS(10000));
    }
}
//------------------------------------------------------------------------------------

//------------------------------------------------------------------------------------
void app_main(void)
{

    //创建事件组
    xEventGroup = xEventGroupCreate();

    if (xEventGroup == NULL)
    {
        printf("xEventGroup Create failed!\n");
    }
    else
    {
        printf("xEventGroup Create success!\n");
        //挂起所有任务
        vTaskSuspendAll();

        xTaskCreate(Task0, "Task0", 1024 * 5, NULL, 1, NULL);
        xTaskCreate(Task1, "Task1", 1024 * 5, NULL, 1, NULL);
        xTaskCreate(Task2, "Task2", 1024 * 5, NULL, 1, NULL);

        xTaskResumeAll();
    }
}


```

# 二十一、Notification Sync 通知同步

我们可以通过任务的通知完成任务的同步，控制任务的运行状态

什么是通知：每一个任务都有一个初始化值为0的32bit通知值。一个任务的通知是阻塞或者解锁任务的事件。

```C
#include <stdio.h>
#include "sdkconfig.h"
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_system.h"
#include "esp_spi_flash.h"

//------------------------------------------------------------------------------------
static TaskHandle_t xTask1 = NULL;
//------------------------------------------------------------------------------------
void Task1(void *pvParameters)
{
    while (1)
    {
        printf("------------------Task1------------------\n");
        printf("Task1 wait notification!\n");

        ulTaskNotifyTake(pdTRUE, portMAX_DELAY);
        printf("------------------------------------\n");
        printf("Task1 got notification!\n");
        vTaskDelay(pdMS_TO_TICKS(3000));
    }
}
//------------------------------------------------------------------------------------
void Task2(void *pvParameters)
{
    while (1)
    {
        vTaskDelay(pdMS_TO_TICKS(5000));

        printf("------------------------------------\n");
        printf("Task2 notify task1!\n");

        xTaskNotifyGive(xTask1);
    }
}
//------------------------------------------------------------------------------------

//------------------------------------------------------------------------------------
void app_main(void)
{

    //挂起所有任务
    vTaskSuspendAll();

    xTaskCreate(Task1, "Task1", 1024 * 5, NULL, 1, &xTask1); //传输Task1的句柄
    xTaskCreate(Task2, "Task2", 1024 * 5, NULL, 1, NULL);

    xTaskResumeAll();
}

```

# 二十二、通知值

## 等待接受通知

```C
BaseType_t xTaskNotifyWait( uint32_t ulBitsToClearOnEntry,    //在进入前清除通知值
                            uint32_t ulBitsToClearOnExit,     //在退出后清除通知值
                            uint32_t *pulNotificationValue,   //将接受的值保存在对应的内存指针当中
                            TickType_t xTicksToWait );
```

## 发送通知

```C
BaseType_t xTaskNotify( TaskHandle_t xTaskToNotify, //通知任务的句柄
                        uint32_t ulValue,           //设置通知的值
                        eNotifyAction eAction );    //设置何时发送通知
```

关于eAction 详情见函数手册的解释

# 二十三、StreamBuffer

![](https://qiniu.yyin.top/StreamBufferTask.png)

用于处理**流数据**StreamData的缓冲，这种流数据不断产生，不断传输，例如**音频数据**。

`xStreamBufferCreate()` Buffer 的创建

```c
#include “FreeRTOS.h”
#include “stream_buffer.h”
StreamBufferHandle_t xStreamBufferCreate( size_t xBufferSizeBytes,     //Buffer的大小
                                          size_t xTriggerLevelBytes ); //触发的大小
```

当Task处于Block状态我们需要发送`xTriggerLevelBytes`大小数据才会UnBlock这个任务

`xStreamBufferSend()` Buffer的发送

```c
size_t xStreamBufferSend( StreamBufferHandle_t xStreamBuffer,   //StreamBuffeer的句柄
                            const void *pvTxData,               //需要发送数据的指针
                          size_t xDataLengthBytes,              //需要发送数据的长度
                          TickType_t xTicksToWait );            //任务阻塞的时间
//返回值是多少数据被写入StreamBuffer
```

`StreamBufferReceive()` Buffer 的接受&#x20;

```java
size_t xStreamBufferReceive( StreamBufferHandle_t xStreamBuffer,
                                       void *pvRxData,   //需要接受数据的指针
                               size_t xBufferLengthBytes,//需要接受数据的长度
                             TickType_t xTicksToWait );  //任务阻塞的时间
```

**实例代码**

```c

#include <stdio.h>

#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "freertos/stream_buffer.h"
#include "esp_system.h"
#include "esp_spi_flash.h"

#include <string.h>

StreamBufferHandle_t StreamBufferHandle = NULL;

//利用Buffer发送数据
void Task1(void *pvParam)
{
    char tx_buf[50];
    int i = 0;
    int str_len = 0;
    int send_bytes = 0;

    while (1)
    {   
        
        
        //字符串格式化命令，param1是需要初始化buffer的指针,param2是我们需要初始化的值,返回初始化后str的长度
        str_len = sprintf(tx_buf, "Hello, Here is YYin %d \n ", i++);

        send_bytes=xStreamBufferSend(StreamBufferHandle,
                            (void *)tx_buf,
                            str_len,
                            portMAX_DELAY);
                             //使得Task2预先进入阻塞
        vTaskDelay(pdMS_TO_TICKS(3000));
    
        printf("=============Send==============!\n");
        
        printf("Send: str_len = %d, send_bytes = %d!\n",str_len,send_bytes);

        


    }

}
//利用Buffer接受数据
void Task2(void *pvParam)
{
    char rx_buf[50];
    int rec_bytes = 0;

    while (1)
    {
        //对于这个buffer进行初始化清零操作,param1:初始化指针;param2:初始化在值;param3:初始化缓冲区的大小
        memset(rx_buf,0,sizeof(rx_buf));

        rec_bytes = xStreamBufferReceive(StreamBufferHandle,
                            (void *)rx_buf,
                            sizeof(rx_buf),
                            portMAX_DELAY);
    
        printf("==============Receive=============!\n");
        printf("Receive: rec_bytes = %d, data = %s\n",rec_bytes,rx_buf);


    }

}

void app_main(void)
{
    StreamBufferHandle = xStreamBufferCreate(1000, 10);

    if (StreamBufferHandle!=NULL)
    {
        vTaskSuspendAll();

        xTaskCreate(Task1, "Task1", 1024 * 5, NULL, 1, NULL);
        xTaskCreate(Task2, "Task2", 1024 * 5, NULL, 1, NULL);

        xTaskResumeAll();
    }
    else
    {
        printf("Fail to create stream buffer!\n");
    }
}


```

```text
=============Send==============!
==============Receive=============!
Receive: rec_bytes = 25, data = Hello, Here is YYin 15

Send: str_len = 25, send_bytes = 25!
=============Send==============!
==============Receive=============!
Receive: rec_bytes = 25, data = Hello, Here is YYin 16

```

## 精确测量Streram Buffer的大小

如何确定一个StreamBuffer的大小呢。如果StreamBuffer太大会浪费系统的资源，如果太小会造成系统工作不正常

我们会创建task3来处理这样的任务，用来监测streamBuffer的空间的变化情况

在初始化的时候我们通常会对string buffer的尺寸设置一个比较大的值

```c
#include “FreeRTOS.h”
#include “stream_buffer.h”
size_t xStreamBufferBytesAvailable( StreamBufferHandle_t xStreamBuffer 
```

这个函数可以让我们监测StringBuffer的空闲空间，可用空间的变化情况，根据返回的数据可以调整流缓冲区的大小

```c
void Task2(void *pvParam)
{
    char rx_buf[50];
    int rec_bytes = 0;

    while (1)
    {
        //对于这个buffer进行初始化清零操作,param1:初始化指针;param2:初始化在值;param3:初始化缓冲区的大小
        memset(rx_buf,0,sizeof(rx_buf));

        rec_bytes = xStreamBufferReceive(StreamBufferHandle,
                            (void *)rx_buf,
                            sizeof(rx_buf),
                            portMAX_DELAY);
    
        printf("==============Receive=============!\n");
        printf("Receive: rec_bytes = %d, data = %s\n",rec_bytes,rx_buf);


    }

}

```

通过这种调试方式我们可以手动调整StreamBuffer的大小初始化值

```c
 StreamBufferHandle = xStreamBufferCreate(200, 10);
```

# 二十四、 Message Buffer

与StreamBuffer不同，MessageBuffer一次只能接收到一条完整的Message。

`xMessageBufferCreate` 创建buffer

```c
#include “FreeRTOS.h”
#include “message_buffer.h”
MessageBufferHandle_t xMessageBufferCreate( size_t xBufferSizeBytes )  //buffe的大小

```

`xMessageBufferReceive` 接受数据

```c
#include “FreeRTOS.h”
#include “message_buffer.h”
size_t xMessageBufferReceive( MessageBufferHandle_t xMessageBuffer, // buffer句柄
                                void *pvRxData,                     //缓冲区的指针
                              size_t xBufferLengthBytes,            //缓冲区的大小
                              TickType_t xTicksToWait );            //等待时间
```

`xMessageBufferSend` 发送数据

```c
#include “FreeRTOS.h”
#include “message_buffer.h”
size_t xMessageBufferSend( MessageBufferHandle_t xMessageBuffer,
                            const void *pvTxData,
                            size_t xDataLengthBytes,
                            TickType_t xTicksToWait );
```

与StreamBuffer不同的是，MessageBuffer会对每个Send都用对应的Receive来接收

# 问题

[Guru Meditation大师冥想](https://blog.csdn.net/weixin_42942530/article/details/103975237)
