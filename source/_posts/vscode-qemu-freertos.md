---
title: Linux搭建FreeRTOS开发环境
date: 2024-02-04 01:18:31
categories:
- FreeRTOS
tags: [VSCode, FreeRTOS, Qemu]
banner_img: /images/vscode-qemu-freertos.jpg
index_img: /images/vscode-qemu-freertos.jpg
excerpt: 基于Linux搭建 VSCode + Qemu + FreeRTOS开发环境
---

{% note success %}
Qemu[^1]是一个通用且开源的虚拟机平台，可以虚拟不同类型的硬件以供软件上的开发调试，比如ARM架构的Cortex-M3系列或者RISC-V架构。本文以Cortex-M3内核为例，基于Ubuntu 20.04(WSL2)搭建FreeRTOS开发环境，演示最基础的Qemu用法。*STM32F103系列的CPU内核就是Cortex-M3*
{% endnote%}

## 1.准备
### 1.1 安装qemu
```sh
sudo apt install qemu-system -y
```
### 1.2 安装arm编译调试工具链
```sh
sudo apt install gcc-arm-none-eabi -y
```
Ubuntu软件默认安装在`/usr/bin`路径下，即**arm-none-eabi-gdb**的位置是`/usr/bin/arm-none-eabi-gdb`，用于后续启动调试。也可以通过命令`which arm-none-eabi-gdb`查询其安装位置。
### 1.3 其余安装
```sh
sudo apt install git libglib2.0-dev libfdt-dev libpixman-1-dev zlib1g-dev ninja-build bison flex libncurses5 libncursesw5-dev -y
```
### 1.4 创建工作区
```sh
mkdir ~/qemu/ && cd ~/qemu
```
### 1.5 下载FreeRTOS
```sh
git clone https://github.com/FreeRTOS/FreeRTOS.git
```
### 1.6 下载FreeRTOS Kernel
```sh
cd FreeRTOS/FreeRTOS/Source && git clone https://github.com/FreeRTOS/FreeRTOS-Kernel
```
### 1.7 移动Kernel到Source目录
{% note warning  %}
该步非常重要，如果`Source`目录下没有FreeRTOS Kernel的源码，则FreeRTOS无法编译运行
{% endnote %}
```sh
cd FreeRTOS-Kernel && mv * ..
```

## 2.VSCode编译运行
### 2.1 进入工作区
```sh
cd ~/qemu/FreeRTOS/FreeRTOS/Demo/CORTEX_MPS2_QEMU_IAR_GCC && code .
```
### 2.2 C/C++配置
`Ctrl + Shift + P`选择`C/C++ Edit Configurations(JSON)`生成C/C++配置，否则运行QEMU报错且VSCode无法正常跳转识别头文件。
{% fold info @c_cpp_properties.json %}
```JSON
{
    "configurations": [
        {
            "name": "Linux",
            "includePath": [
                "${workspaceFolder}/**",
                "~/qemu/FreeRTOS/FreeRTOS/Source/**",
                "~/qemu/FreeRTOS/FreeRTOS/Demo/Common/**"
            ],
            "defines": [],
            "compilerPath": "/usr/bin/gcc",
            "cStandard": "c17",
            "cppStandard": "gnu++14",
            "intelliSenseMode": "linux-gcc-x64"
        }
    ],
    "version": 4
}
```
{% endfold %}

### 2.3 调试配置
确保`launch.json`文件中的`miDebuggerPath`为`arm-none-eabi-gdb`的安装路径
{% fold info @launch.json%}
```JSON
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Launch QEMU RTOSDemo",
            "type": "cppdbg",
            "request": "launch",
            "program": "${workspaceFolder}/build/gcc/output/RTOSDemo.out",
            "cwd": "${workspaceFolder}",
            "miDebuggerPath": "/usr/bin/arm-none-eabi-gdb",
            "miDebuggerServerAddress": "localhost:1234",
            "stopAtEntry": true,
            "preLaunchTask": "Run QEMU"
        }
    ]
}
```
{% endfold %}

### 2.4 启动调试

`F5`启动调试(自动编译)，断点自动暂停在main入口处，再按`F5`运行程序，切换`TERMINAL`选项卡查看打印情况

![F5启动调试](start_debug.gif)

{% note warning %}
如果arm-none-eabi-gcc提示*error while loading shared libraries: libncurses.so.5: cannot open shared object file: No such file*，则为`libncurses.so.5`创建软连接
{% endnote %}
查找`/usr`目录下所有`libncursesw.so`文件
```sh
find /usr -name "libncursesw.so*"
```
创建软连接
```sh
ln -s /usr/lib/x86_64-linux-gnu/libncursesw.so.6.2 /usr/lib/x86_64-linux-gnu/libncursesw.so.5
```

## 3.程序分析
该FreeRTOS的Demo创建了两个任务，一个定时器以及一个先进先出队列
- 发送任务`prvQueueSendTask`，每200ms往队列`xQueue`发送一个消息
- 定时器`xTimer`设置为**自动重载**，每2000ms进入一次定时器回调，在回调中往队列`xQueue`发送一个消息
- 接收任务`prvQueueReceiveTask`，循环等待队列`xQueue`的消息，收到消息则判断消息的类型并进行打印

![程序分析](main_blinky.png)

## 4.增加日志打印
复制`debug_log.h`和`debug_log.c`到Demo根目录，文件来源于[终端彩色打印](https://blog.gogo.uno/2024/02/12/c-color-print/)
1. 在`main_blinky.c`文件中导入头文件`#include "debug_log.h"`
2. 将`prvQueueReceiveTask`函数中的`printf`更换成`DBG_LOGI`和`DBG_LOGW`，运行效果如下

![彩色日志打印效果](color_log.png)

## 5.内存泄露分析
在`prvQueueReceiveTask`函数里的`if( ulReceivedValue == mainVALUE_SENT_FROM_TASK )`条件，该条件接收来自发送队列发送的消息，将该条件下的内容修改为以下的代码，分析内存泄露的情况

```c
// 打印队列接收的消息和当前的可用的堆空间
DBG_LOGI( "Message received from task, FreeHeap = %d", xPortGetFreeHeapSize());
// 申请一个Char空间
char *ch = (char *)pvPortMalloc(sizeof(char));
// 释放空间
vPortFree(ch);
```

![没有发生内存泄露](no_memory_leak.png)

从上面打印可用的堆空间来看，程序没有发生内存泄露。如果只申请不释放，也就是注释`vPortFree(ch);`这句代码，重新编译运行应该能够看到**可用堆空间不断减少**，也就是发生了**内存泄露**

![发生内存泄露的打印](leak_terminal.gif)

在打印处添加记录点`Add Logpoint`, 输入以下表达式，同样可以判断出内存泄露。
```sh
FreeHeap = {xPortGetFreeHeapSize()}
```
![控制台监控记录点](leak_console.gif)

增大申请的空间，能够更快地观察到内存泄露到最后的结果
```c
char *ch = (char *)pvPortMalloc(sizeof(char) * 1024 * 10);
```
![内存泄露最终结果](leak_chunk.gif)

## 6.定时器分析
在`prvQueueReceiveTask`函数里的`if( ulReceivedValue == mainVALUE_SENT_FROM_TIMER )`条件，该条件接收来自定时器中断发出的消息，将该条件下的内容改为以下的代码，粗略分析定时器的重载时间
```c
// 定义上次消息达到的时间(静态定义，因此只会初始化一次)
static int last_time = 0;
// 获取当前时间(1tick = 1ms)并减去上次记录的时间，也就是定时器的重载时间
int actual_period = xTaskGetTickCount() - last_time;
// 打印定时器信息
DBG_LOGW( "Message received from software timer, period = %d", actual_period);
// 保存当前时间
last_time = xTaskGetTickCount();
```

![定时器分析](timer.gif)

根据Watch窗口的值以及终端打印的值，任务接收到定时器消息的间隔是1999ms或2000ms

## 7.cJSON内存管理
{% cb FreeRTOS添加cJSON库，并尝试分析内存泄露的状况, true, false, false %}
{% note primary %}
cJSON是一个非常流行的轻量级的JSON解析器，只需要一个头文件和源文件就可以使用。但是其节点的创建以及回收默认使用的是C语言中的`malloc`和`free`，随着程序逻辑复杂度增加，一旦节点没有被正确回收就会发生内存泄露的情况，且难以定位调试。通过FreeRTOS的堆内存管理的方式，可以有效判断在使用cJSON时是否发生了内存泄露。
{% endnote %}

### 7.1 导入cJSON文件
根据该cJSON官方Github地址[^4]下载`cJSON.h`和`cSJON.c`文件
- 复制到`main_blinky.c`同级目录下
- 在`Makefile`文件中，增加编译选项`-lm`且将`cSJON.c`加入编译
- 禁用`malloc`警告函数，否则编译不通过
- 增加任务深度，否则调用cJSON函数极容易出现`Stack overflow in Rx`的栈溢出错误

![cJSON.c加入编译](cjson_compile.png)

![增加编译选项](add_compile_option.png)

![禁用malloc警告](disable_malloc_warning.png)

![增加任务深度](enlarge_task_depth.png)

### 7.2 注册FreeRTOS内存管理函数
在实际使用cJSON功能函数之前，将FreeRTOS的内存管理函数注册给cJSON的内存管理函数，则通过检测`xPortGetFreeHeapSize()`函数返回值就能够快速判断出cJSON是否存在内存泄露。
```c
#if 1
    cJSON_Hooks freertos_hooks = {0};
    freertos_hooks.malloc_fn = pvPortMalloc;
    freertos_hooks.free_fn = vPortFree;
    // 初始化CJSON内存管理函数
    cJSON_InitHooks(&freertos_hooks);
#endif
```

### 7.3 测试情况
#### 7.3.1 根节点
创建一个root节点，分别测试是否执行了`cJSON_Delete`函数时的内存情况
![没有删除root节点，发生内存泄露](not_delete_root.png)
![删除root节点，没有发生内存泄露](delete_root.png)

#### 7.3.2 测试cJSON_Print函数
创建一个JSON，包含一个字符串的键值对，分别测试在调用`cJSON_PrintUnformatted`之后是否释放该字符串指针的内存情况
![不释放字符串指针，发生内存泄露](not_free_cjson_string.png)
![释放字符串指针，没有发生内存泄露](free_cjson_string.png)

#### 7.3.3 使用原生内存管理函数
在不执行`cJSON_Delete`函数下，分别测试在启用以及禁用FreeRTOS的内存管理函数的内存情况

![使用FreeRTOS内存管理函数可以判断出内存泄露](visible_mem_leak.png)
![使用C语言原生的内存管理函数无法判断出内存泄露](invisible_mem_leak.png)

## 参考
[^1]: [Qemu官网](https://www.qemu.org)
[^2]: [QEMU+FreeRTOS+STM32+VS Code](https://blog.csdn.net/hihan_5/article/details/130594536)
[^3]: [qemu+vscode+FreeRTOS的demo测试及复现](https://blog.csdn.net/u010829693/article/details/132160976)
[^4]: [cJSON官方Github地址](https://github.com/DaveGamble/cJSON)
