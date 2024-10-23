---
title: STM32-FreeRTOS移植与调试
date: 2024-03-24 07:41:27
categories:
- STM32
tags: [STM32, C/C++, FreeRTOS]
banner_img: /images/stm32-freertos.jpg
index_img: /images/stm32-freertos.jpg
excerpt: 使用STM32CubeMX生成FreeRTOS中间件以及调试FreeRTOS
---

{% note success %}
使用STM32CubeMX生成FreeRTOS中间件并创建两个任务，一个任务循环闪灯，另一个任务循环打印串口消息。进阶使用`XRTOS`调试，监控FreeRTOS任务堆栈大小以及分析任务的CPU使用率。
{% endnote %}

## 1.初始化工程
### 1.1 新建工程
STM32CubeMX创建工程，系统时钟、时钟树配置和调试口自行配置。PC13设置输出，启用USART1(提前用USB转串口模块接入PA9和PA10)，工程名称为`freertos_demo`。有问题则参考之前的文章。

![新建工程](new_project.png)

### 1.2 配置FreeRTOS

- 展开`Middleware and Software Packs`选项卡
- 选择`FreeRTOS`，Interface选择`CMSIS_V1`，因为STM32F103C8T6就是**M3**内核的芯片，选择`CMSIS_V1`会更节省空间。

![选择FreeRTOS配置](select_config.png)

### 1.3 创建任务
- 选择`Tasks and Queues`标签，点击`Add`按钮
- 将`Task Name`修改为**LedTask**
- 和`Entry Function`均修改为**led_task**
- 点击`OK`按钮确认
- 同理再创建一个串口任务`uart_task`

![创建闪灯任务](new_led_task.png)

![创建串口任务](new_uart_task.png)


### 1.4 编译导入

#### 1.4.1 Keil编译 
点击`GENRATE CODE`生成工程代码，点击`Open Project`,尝试用Keil编译，没有报错则可以将工程导入EIDE

![生成工程代码](generate_code.gif)

{% note warning %}
如果STM32CubeMX生成工程之后，**KEIL没有编译**，则EIDE尝试导入MDK工程时，可能会出现编译失败的情况，比如丢失RAM/ROM布局导致的编译失败
{% endnote %}

#### 1.4.2 EIDE编译
打开VSCode，EIDE插件选择`Import Project`,选择刚刚创建的工程, 注意EIDE的工作区**不要**保存在与MDK同一目录下，虽然不影响EIDE编译调试，但是会导致VSCode工作区无法跟踪工程根目录的文件。**EIDE工作区需要保存在工程根目录下**

![EIDE导入](import_project.gif)

![EIDE编译烧录](eide_build_flash.gif)

## 2.编辑任务
### 2.1 闪灯任务
- `main.c`文件中的`MX_FREERTOS_Init();`函数执行初始化任务，跳转进入该函数
- 往下滑找到`led_task`任务，跳转进入该任务
- 将原本任务里的`for`循环中的`osDelay(1);`修改为以下代码
```c
osDelay(500);
HAL_GPIO_TogglePin(GPIOC, GPIO_PIN_13);
```

![闪灯任务](led_task.png)

### 2.2 串口任务

根据[STM32-串口USART](https://blog.gogo.uno/2024/03/02/stm32-uart/)的第八点**重定向printf**，成功配置串口重定向之后尝试在串口任务每隔500ms打印数据

![串口任务](uart_task.png)

## 3.调试FreeRTOS
{% note primary %}
在启动任何调试之前都必须确保工程的**优化等级为0**，否则各种情况都有可能发生，比如：创建了断点，但是程序无法暂停在断点处；或者创建断点的时候断点发生偏移。注意，如果EIDE**再次导入**STM32CubeMX生成的MDK工程，则需要**再次检查**优化等级。
{% endnote %}


![确保优化等级为0](compile_level.png)

`F5`尝试启动调试，如果可以正常启动调试则在程序运行后几秒后创建一个断点，点击底部栏的**XRTOS**面板，查看任务状态

![XRTOS查看任务状态](xrtos_debug.png)

XRTOS的面板必须在**程序暂停后才能更新**，观察XRTOS的任务状态列表，TaskName表示任务名称，黄色高亮的行代表当前断点暂停在哪个任务，任务优先级，程序地址等。但是其中有一些问题，比如Stack End、Stack Used以及RunTime等都是以问号显示。需要配置更多宏以及定时器资源才能正确监控任务状态。

### 3.1 启动任务监控
在`FreeRTOSConfig.h`文件中，**增加**宏`configUSE_TRACE_FACILITY`
```c
/* USER CODE BEGIN Defines */
/* Section where parameter definitions can be added (for instance, to override default ones in FreeRTOS.h) */
#define configUSE_TRACE_FACILITY 1
/* USER CODE END Defines */
```

编译后启动调试，应该就能看到插件给出的提示, 单击展开提示

![监控提示](debug_hint.png)

### 3.2 开启栈尾监控
在`FreeRTOSConfig.h`文件中，**增加**宏`configRECORD_STACK_HIGH_ADDRESS`，尽可能确保FreeRTOS的版本在V10.0.0以上
```c
/* USER CODE BEGIN Defines */
/* Section where parameter definitions can be added (for instance, to override default ones in FreeRTOS.h) */
#define configUSE_TRACE_FACILITY                1
#define configRECORD_STACK_HIGH_ADDRESS         1
/* USER CODE END Defines */
```

编译后启动调试，等任务运行几秒后创建断点暂停程序，查看任务状态，应该可以看到栈尾状态，也就能查看完整的任务深度状态

![任务深度](stack_depth.png)

### 3.3 统计CPU使用率
{% note primary %}
CPU使用率，这里称为`Runtime`，用于统计每个任务的CPU使用率，能够直观地判断各个任务对CPU的占用情况。但是获取该`Runtime`属性需要消耗一个高速定时器。
{% endnote %}

根据监控提示，用于统计CPU使用率的高速定时器必须10倍与FreeRTOS的系统中断时间（默认是1ms）。因此定时器的中断时间至少在100us以内，等价于10kHZ频率。STM32的定时器TIM1能够符合要求。

>  Also, add the following two macros to provide a high speed counter -- something at least 10x faster than your RTOS scheduler tick. One strategy could be to use a HW counter and sample its current value when needed

STM32CubeMX配置STM32的TIM1频率为1Mhz，溢出计数为100-1，实现弱函数`HAL_TIM_PeriodElapsedCallback`并在该函数里递增计数，并将该计数值设置为全局变量以供FreeRTOS调用，就能够实现统计CPU使用率的效果。

![创建定时器TIM1](create_tim1.png)

![启用定时器中断](tim1_intr_enable.png)

`main.c`中添加变量`CPU_Runtime`并初始化为0
```c
/* USER CODE BEGIN PV */
volatile uint32_t CPU_RunTime = 0;
/* USER CODE END PV */
```

`main.c`中启用定时器TIM1中断
```c
/* USER CODE BEGIN 2 */
HAL_TIM_Base_Start_IT(&htim1);
/* USER CODE END 2 */
```

`main.c`中实现`TIM1`中断，累加`CPU_RunTime`
```c
/* USER CODE BEGIN 4 */
void HAL_TIM_PeriodElapsedCallback(TIM_HandleTypeDef *htim)
{
    if (htim == &htim1)
    {
        CPU_RunTime++; // 更新计数值
    }
}
/* USER CODE END 4 */
```
`FreeRTOSConfig.h`中添加外部变量`CPU_Runtime`以及采样时间的宏, 完整FreeRTOS调试配置如下
```c
/* USER CODE BEGIN Defines */
/* Section where parameter definitions can be added (for instance, to override default ones in FreeRTOS.h) */
#define configUSE_TRACE_FACILITY                 1
#define configRECORD_STACK_HIGH_ADDRESS          1
extern volatile uint32_t CPU_RunTime;
#define configGENERATE_RUN_TIME_STATS            1
#define portCONFIGURE_TIMER_FOR_RUN_TIME_STATS() (CPU_RunTime = 0ul)
#define portGET_RUN_TIME_COUNTER_VALUE()          CPU_RunTime
/* USER CODE END Defines */
```
编译启动调试后，能够正常看到各个任务的CPU使用率

![CPU使用率](runtime.png)

## 4.监控分析
### 4.1 任务深度
以闪灯任务`LedTask`为例，在未申请的临时变量时，其内存使用是**96Byte**，而在申请了临时变量后，内存使用是**352Byte**，相差刚好是**256Byte**。

![未申请临时变量](raw_led_task.png)

![已申请临时变量](chunk_led_task.png)

### 4.2 CPU使用率
- `LedTask`闪灯任务中增加阻塞延时500ms`HAL_Delay(500)`
- `UartTask`串口任务注释`printf`语句，修改延迟时间为1000ms,并增加切换LED状态代码

此时两个任务执行的内容基本一致：都是延迟1s切换LED灯状态。唯一的区别就是闪灯任务的延迟是HAL库进行阻塞延时。进入调试状态，能够看到闪灯任务的CPU占用率是45.77%，接近50%；而串口任务占用率是0%。

![阻塞延迟对CPU占用率的影响](delay_runtime.png)

删除闪灯任务中的延时，循环中仅切换LED灯，然后再加入`osDelay(1);`，调试查看前后任务的CPU占用率时间区别，明显能够看出加入了1ms延时的任务的CPU使用率有了明显的降低。

![循环中没有延时](non_delay.png)

![循环中包含1ms延时](delay_1ms.png)

修改串口任务延时1ms再打印数据，对比闪灯任务能够明显看到`printf`函数执行是相当耗时的。

![串口对比闪灯任务的CPU使用率](serial_to_led.png)

## 参考
[^1]: [CPU使用率统计](https://doc.embedfire.com/rtos/freertos/zh/latest/application/cpu_usage_rate.html#)




