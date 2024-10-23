---
title: Windows搭建STM32开发环境(基础篇)
date: 2024-02-13 00:00:00
categories:
- STM32
tags: [STM32, VSCode, C/C++]
banner_img: /images/vscode-stm32-develop.jpg
index_img: /images/vscode-stm32-develop.jpg
excerpt: 搭建VSCode + CubeMX +  STM32开发环境
---

{% note success %}
在Windows下通过VSCode的插件`Keil Assistant`实现**编译**以及**烧录**STM32工程。注意，基础篇**不包含**调试功能，进阶篇[^1]才提供调试功能。虽然缺少调试功能，但是该插件的配置非常简单，且调用的是Keil进行的编译下载，理论上没有兼容性的问题。
{% endnote %}

## 1.准备
### 1.1 硬件
- STM32F103C8T6开发板，确保开发板上的**PC13**引脚连接LED灯
- JLink调试器
- 按照SWD方式接线，总共4根线：VCC(3.3V), SWD, SWCLK, GND
### 1.2 软件
- 安装Keil MDK，自行确保软件安装成功[^2]
- 安装STM32CubeMX（如果STM32CubeMX无法生成MDK-ARM的代码，可以尝试安装旧版的CubeMX）
### 1.3 VSCode插件
#### 1.3.1 安装Keil Assistant插件
![Keil Assistant插件](keil_assistant.png)
#### 1.3.2 Keil Assistant配置
`Keil Assistant.MDK: Uv4 Path`处填入Keil的安装路径,也可以顺便把`C51`的路径填了。
![Keil Assistant配置](keil_assistant_setting.gif)

## 2.CubeMX生成MDK工程
生成一个闪灯程序[^3]，可以按照以下步骤生成一个STM32的最小工程
### 2.1 新建工程
![新建工程](new_project.png)
### 2.2 选择目标芯片
`PartNumber`处填入**STM32F103C8**，然后选择目标芯片，再点击`Start Project`
![选择目标芯片](target_chip.png)
### 2.3 时钟配置
展开`System Core`选项卡，选择`RCC`，`High Speed Clock(HSE)`下拉框选择`Crystal/Ceramic REsonator`
![选择外部时钟](rcc.png)
### 2.4 Debug配置
展开`System Core`选项卡，选择`SYS`，`Debug`下拉框选择`Serial Wire`
![Debug配置](sys_debug.png)
### 2.5 配置外设
在此处配置一个GPIO的输出，即PC13
![GPIO配置](gpio_config.gif)
### 2.6 配置时钟树
点击`Clock Configuration`选项卡，先选择`PLLCLK`，再选择`HSE`，最后在HCLK(MHz)中填入72并**回车确认**
![时钟树配置](clock_tree.gif)
### 2.7 生成MDK工程
- 选择`Project Manager`选项卡
- 侧边栏`Project`
  - `Project Name`填入项目名称**blink**
  - `Toolchain / IDE`选择**MDK-ARM**，并确保`Min Version`大于**V5**
- 侧边栏`Code Generator`
  - 选择`Copy only necessary library files`
  - 勾选`Generate peripheral initialization as a pair of '.c/.h' files per peripheral` 
- 最后点击顶部`GENERATE CODE`按钮生成MDK工程

![生成MDK工程](generate_mdk.gif)

## 3.MDK编译下载
### 3.1 打开MDK工程
可以在CubeMX生成工程后，点击`Open Project`打开工程，也可以找到实际存储该工程的路径并打开。
![打开MDK工程](open_mdk.png)
### 3.2 选择JLink为下载调试工具
需要确保JLink的下载方式为**SWD**
![选择JLink](choose_jlink.gif)
### 3.3 设置下载完自动重启
![下载完自动重启](auto_reset.gif)

### 3.4 编译
在`main.c`的while循环中增加一个切换高低电平输出，并延迟500ms。点击编译。
```c
// LED灯以500毫秒的间隔闪烁
HAL_GPIO_TogglePin(GPIOC, GPIO_PIN_13);
HAL_Delay(500);
```
![MDK编译](mdk_compile.png)

### 3.5 下载
![MDK下载](mdk_flash.png)
![下载运行结果](flash_to_stm32.gif)

## 4.Keil Assistant编译下载
### 4.1 打开MDK工程
- 点击`Keil UVISION PROJECT`最右侧的按钮: `Open keil uvision project`
- 找到`blink.uvprojx`文件并导入
- 右下角点击`Ok`切换工作区

![Keil_Assistant-打开MDK工程](keil_assistant_open_mdk.gif)

{% note warning%}
切换工作区后，Keil Assistant插件可能需要花几秒的时间才会把工程加载出来。
{% endnote %}

### 4.2 编译
```c
// 修改为100ms，LED灯以100毫秒的间隔闪烁
HAL_Delay(100);
```
![Keil_Assistant-编译](keil_assistant_compile.png)

### 4.3 下载
![Keil_Assistant-下载](keil_assistant_flash.png)

## 5.注意事项
### 5.1 无法烧录问题排查
{% note warning%}
如果编译正确却无法烧录程序，说明没有正确建立SWD连接。先判断电脑是否与JLink建立连接，如果电脑识别不到JLink可以尝试更换USB数据线(尽可能选择质量好的USB数据线)。如果电脑可以检测到JLink，但是Keil里无法识别SW设备，则检查SWD的接线是否正确，以及接线是否牢固。**切记不要接错VCC和GND**
{% endnote %}

![JLink与电脑正确连接](jlink_computer.png)
![JLink与STM32正确连接](jlink_stm32.png)

### 5.2 VSCode头文件错误问题
{% note warning %}
在VSCode如果出现头文件报错等问题，记得先检查`c_cpp_properties.json`文件的`includePath`是否包含了正确以及完整的库（虽然并不影响编译），解决此问题的方法可以全局搜索具体报错的头文件，找到其位置。请不要在VSCode里手动添加路径到`c_cpp_properties.json`文件，因为每编译一次就会此文件就会被覆盖掉。正确的做法是**打开Keil**去添加**C/C++选项卡**的`includePath`，然后再编译一下就可以自动加载到正确的库文件（因为VSCode每次编译都会读取一遍Keil的配置）
{% endnote %}

![增加includePath以正常跳转](path_not_found.gif)

## 参考
[^1]: [Windows搭建STM32开发环境-进阶篇](https://blog.gogo.uno/2024/02/13/vscode-stm32-develop-pro/)
[^2]: [安装Keil MDK软件](https://blog.csdn.net/Mculover666/article/details/89469764)
[^3]: [STM32CubeMX生成闪灯工程](https://blog.csdn.net/h568630659/article/details/121404155)
[^4]: [VS Code环境下编辑、编译、下载Keil工程代码](https://mp.weixin.qq.com/s/acDaKenu6yZU_INZQ0NhZw)
[^5]: [Vscode 编译Keil ARM 工程 出现 未定义标识符的解决办法](https://blog.csdn.net/m0_46556792/article/details/107335638)
