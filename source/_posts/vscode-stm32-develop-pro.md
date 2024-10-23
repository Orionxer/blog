---
title: Windows搭建STM32开发环境(进阶篇)
date: 2024-02-14 00:45:22
categories:
- STM32
tags: [STM32, VSCode, C/C++]
banner_img: /images/vscode-stm32-develop-pro.jpg
index_img: /images/vscode-stm32-develop-pro.jpg
excerpt: 搭建VSCode + EIDE + Cortex Debug + STM32开发环境
---

{% note success %}
在Windows下通过VSCode的`EIDE`和`Cortex-Debug`插件实现**编译、烧录、调试**STM32。不仅兼容MDK工程，且具有自己的独立的工作区，不影响原来的MDK工程。*本质上该教程适用所有ARM架构与C51架构的芯片，甚至是一些RISC-V架构*。也可以通过EIDE的官方教程[^1]搭建开发环境。
{% endnote %}

## 1.准备
- 确保JLink与STM32建立了正确的SWD连接
- 确保MDK能够正确编译烧录STM32工程

{% note warning %}
如果无法确保以上两点请参考基础篇[^2]的细节。
{% endnote %}

## 2.插件安装
### 2.1 安装EIDE
![安装EIDE](eide.png)
### 2.2 安装Cortex-Debug
![安装Cortex-debug](cortex_debug.png)

## 3.EIDE插件配置
### 3.1 设置编译器工具链
VSCode侧边栏切换到EIDE选项卡，点击`Configure Toolchain`，设置EIDE的编译器路径(该插件需要使用Keil内置的编译器)。假设Keil安装的位置是`D:\Keil_v5`，则选择的文件夹路径应该为`D:\Keil_v5\ARM\ARMCC\bin`,*大小写不敏感*

![设置EIDE的编译工具链](eide_toolchain.png)

### 3.2 axf文件转elf文件
打开`Open plug-in Settings`，向下找到`EIDE.ARM.Option: Axf To Elf`并勾选上。
![axf文件转elf文件](axf_to_elf.png)
## 4.EIDE使用入门
{% note primary %}
强烈建议先读一遍官方文档[^1]再按照以下步骤操作以降低失败的概率。
{% endnote%}
### 4.1 导入MDK项目
选择`Import Project`，再选择`MDK`，再选中`blink.uvprojx`，再点击右下角的`ok`保存EIDE工作区到当前工程目录下
![导入MDK项目](import_project.gif)
### 4.2 芯片支持包
根据提示选择目标为STM32F1C8T6的芯片支持包。像STM32等比较知名的芯片可以直接在网络上下载`From Repo`；如果是华大的芯片比如HC32系列，则需要从本地`From Disk`读取.pack后缀的芯片支持包。其余同理，本质上需要提供`.FLM`文件以及`.svd`文件才能够正常烧录调试
![芯片支持包](package_support.gif)
### 4.3 构建配置
一般在导入MDK项目时，EIDE插件就会复制该工程的构建配置。除非在编译不通过的时候，则逐项检查构建配置的每一项是否与原来的MDK项目一致。
#### 4.3.1 RAM/ROM Layout
以RAM/ROM Layout为例，EIDE会保持与原MDK工程一致
![RAM/ROM Layout对比](ram_rom.png)
#### 4.3.2 构建器选项
构建器选项中有一个比较重要的设置就是**编译优化等级**,即**Optimization Level**，在项目开发阶段尽量设置为`-O0`，也就是不开启任何的优化以方便调试。
![设置编译优化登记为-O0](optimization_level.png)
{% note warning %}
如果在调试中碰到异常情况：比如断点明明设置好了，但是断点无法暂停在指定位置；或者断点在设置之后偏移了很远。通常是由于编译器默认开启了`O3`的优化导致的。
{% endnote %}
### 4.4 烧录配置
![烧录配置选择芯片](flash_config.png)
### 4.5 编译
点击`Build`按钮，或者快捷键`F7`，EIDE会自动保存当前工程文件并开始编译，`Terminal`窗口会输出编译信息且包含了**RAM/ROM的静态使用情况**，在资源使用评估方面非常直观🎉。
![EIDE编译](eide_build.png)
### 4.6 烧录
![EIDE烧录](eide_flash.png)

## 5.Cortex-Debug插件配置
{% note primary %}
Cortex-Debug插件必须正确配置**armToolchainPath**以及**JLinkGDBServerPath**才能启动调试。如果启动调试失败就去`Debug Console`控制台定位具体的报错原因。
{% endnote %}
### 5.1 下载安装ARM工具链
从ARM官网下载ARM工具链[^3]，下载后安装，并复制`bin`路径(目的是为了告诉Cortex-Debug插件在启动调试的时候找到`arm-none-eabi-gdb.exe`的位置)

![ARM官网选择工具链的版本](arm-gun-toochain.png)

![安装成功时gdb的位置](arm-gdb-location.png)

{% note info %}
以安装路径`D:\arm`为例， 则需要复制的路径应该为`D:\arm\13.2 Rel1\bin`，因为Windows文件系统的原因需要多加入一个斜杠，填入Cortex-Debug的配置时，应为`D:\\arm\\13.2 Rel1\\bin`
{% endnote %}

### 5.2 设置方法
- 找到Cortex-Debug插件，右键点击`Extension Settings`
- 找到`Cortex-debug: Arm Toolchain Path`点击`Edit in setting.json`
- 在`cortex-debug.armToolchainPath`处粘贴以上准备好的ARM工具链路径
- 检查`cortex-debug.JLinkGDBServerPath`路径是否正确(比如*Administrator*就可能需要换成自己的用户名，尽量保证不要有中文)
以下提供一个示例配置
```json
"cortex-debug.armToolchainPath": "D:\\arm\\13.2 Rel1\\bin",
"cortex-debug.JLinkGDBServerPath": "C:\\Users\\Administrator\\.eide\\tools\\jlink\\JLinkGDBServerCL.exe",
```
![设置Cortex-Debug插件](debug_config.gif)

## 6.调试
### 6.1 启动调试
`F5`启动调试，该调试启动的时间较长，实测需要**20秒**的时间才能进入主程序，不过在调试过程并不慢。调试的功能与逻辑均与VSCode一致。关于调试的入门用法可以参考《Linux搭建C开发环境》的*4.调试入门*[^4]
![启动调试](debug.gif)
### 6.2 查看寄存器
以本篇文章的`blink`工程为例，不断翻转PC13的GPIO高低电平以实现LED灯闪烁。根据STM32F103的数据手册，其对应的`GPIOC`组下的`ODR`寄存器功能，`Bit 13`的值应该不断在`0`和`1`之前切换
![查看寄存器的值](register.gif)
### 6.3 Live Watch
MEMORY窗口中提供了`Live Watch`的功能，与Keil的`Watch`功能一致，能够在不暂停程序的情况下，动态监控变量的值。添加到Live Watch的变量必须为**全局变量**，否则其无法监控到。
启用该功能前需要在`Launch.json`中添加`liveWatch`键值对，以下提供一个参考
{% fold info@launch.json%}
```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "cwd": "${workspaceRoot}",
            "type": "cortex-debug",
            "request": "launch",
            "name": "jlink",
            "servertype": "jlink",
            "interface": "swd",
            "executable": "build\\blink\\blink.elf",
            "runToEntryPoint": "main",
            "device": "STM32F103C8",
            "svdFile": ".pack/Keil/STM32F1xx_DFP.2.3.0/SVD/STM32F103xx.svd",
            "liveWatch": {
                "enabled": true,
                "samplesPerSecond": 4
            }
        }
    ]
}
```
{% endfold %}
![动态监控变量的值](live_watch.gif)
### 6.4 内存表
MEMORY窗口中提供了`MEMORY`功能，即内存表，输入合法地址就能监控其内存的值，甚至可以查看其内存值对应的ASCII码，设置内存的大小端显示顺序，以及根据需要按照4-Byte或者8-Byte对齐显示。以下写了一个长度为100的数组array，每循环一次所有数组的值自加1，内存发生变化的地方均会高亮显示。
![内存表](memory.gif)

## 参考
[^1]: [Embedded IDE官方教程](https://em-ide.com/zh-cn/docs/intro)
[^2]: [Windows搭建STM32开发环境-基础篇](https://blog.gogo.uno/2024/02/12/vscode-stm32-develop/)
[^3]: [ARM工具链官方下载页面](https://developer.arm.com/downloads/-/arm-gnu-toolchain-downloads)
[^4]: [Linux搭建C开发环境](https://blog.gogo.uno/2024/02/04/vscode-develop-c/)