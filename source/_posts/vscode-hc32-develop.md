---
title: VSCode编译烧录调试HC32F460x
date: 2024-09-04 01:29:01
categories:
- Other
tags: [VSCode, HC32]
banner_img: /images/vscode-hc32-develop.jpg
index_img: /images/vscode-hc32-develop.jpg
excerpt: 通过VSCode插件EIDE以及Cortex-Debug实现华大HC32F460x芯片编译烧录调试
---

{% note success %}
通过VSCode的**EIDE**和**Cortex-Debug**插件实现**编译、烧录、调试**华大HC32F460x芯片。EIDE具有独立的工作区，完全兼容MDK工程。分别使用JLink和DapLink进行烧录调试。
{% endnote %}

> 华大HC32F460x使用的是ARM Cortex M4架构，因此只要配置了正确的芯片支持包，就能够实现与STM32系列芯片一致的开发效果。如果不熟悉EIDE和Cortex Debug的使用方法，建议先阅读**Windows搭建STM32开发环境-进阶篇**[^1]和**cortex-debug 用法**[^2]。

## 1.编译
### 1.1 导入Keil项目
准备好能够编译成功的Keil项目，VSCode工作区下点击`Import Project`，选择**MDK**，选择Keil项目。
![导入Keil项目](import_project.gif)

### 1.2 加载芯片支持包
点击**Chip Support Package**，选择**From Disk**，选择华大提供的芯片支持包(.pack后缀)，再根据自己的芯片选择具体的型号，比如我这里是`HC32F460KEUA`
![加载芯片支持包](install_package.gif)

### 1.3 编译
点击**Build**，快捷键**F7**，编译成功后会输出已使用的静态RAM/ROM信息。其他编译输出信息参考EIDE官方文档[^3]说明。
![编译结果](compile.png)

## 2.JLink烧录调试
### 2.1 添加新设备
{% note info %}
JLink默认的设备列表不包含HC32F460x，因此需要手动添加新设备给JLink的设备列表。
{% endnote %}
- 找到EIDE的JLink安装目录，比如`C:\Users\Orionxer\.eide\tools\jlink`，备份JLinkDevices.xml文件
- 进入Devices文件夹，**新建HDSC文件夹**
- 解压Pack包后找到**HC32F460_512K.FLM**文件，将文件复制到HDSC文件夹下，将**HC32F460_512K.FLM**重命名为**HC32F46x.FLM**
- 编辑JLinkDevices.xml文件，复制以下内容，**在文件底部添加**新设备
{% fold info@JLinkDevices.xml%}
```xml
<!--                 -->
<!-- HDSC (HC32) -->
<!--                 -->
<Device>
   <ChipInfo Vendor="HDSC" Name="HC32F46x"  WorkRAMAddr="0x20000000" WorkRAMSize="0x10000" Core="JLINK_CORE_CORTEX_M4"/>
   <FlashBankInfo Name="Flash_512K" BaseAddr="0x0" MaxSize="0x80000" Loader="Devices/HDSC/HC32F46x.FLM" LoaderType="FLASH_ALGO_TYPE_OPEN" AlwaysPresent="1"/>
</Device>
```
{% endfold %}
![添加新设备到JLink设备列表](add_new_device.png)

- VSCode重新加载JLink设备：`Ctrl + Shift + P`打开命令面板，输入**Reload Jlink Devices List**即可
![重新加载JLink设备列表](reload_jlink_devices.png)

{% note warning %}
如果JLink弹窗提示选择设备，注意华大的缩写是HDSC，输入HC32是找不到的
{% endnote %}

### 2.2 烧录
点击**Flasher Configurations**的切换按钮，确保烧录方式为**JLink**，选择**CPU Name**为**HC32F460x**，点击**Program Flash**按钮尝试烧录
![JLink烧录](jlink_flash.gif)

### 2.3 调试
#### 2.3.1 Cortex Debug配置
自行确保Cortex Debug插件配置正确，以下提供一个示例配置，请根据自己的实际情况进行修改
{% fold info@settings.json%}
```json
"cortex-debug.armToolchainPath": "D:\\Program Files (x86)\\Arm GNU Toolchain arm-none-eabi\\12.2 mpacbti-rel1\\bin",
"cortex-debug.JLinkGDBServerPath": "C:\\Users\\Orionxer\\.eide\\tools\\jlink\\JLinkGDBServerCL.exe",
```
{% endfold %}

#### 2.3.2 生成调试配置
- EIDE中**右键点击**项目，选择**Generate Debugger Configuration**，选择**JLink**，
- 一般默认参数不需要修改，点击**Create**
- **F5**启动调试，调试启动较慢，大概耗时10-20秒不等

![JLink调试](jlink_debug.gif)

关于调试的更多用法，可以参考Linux搭建C开发环境的[^4]**第4点-调试入门**，详细介绍了各类断点、监控变量、调用堆栈以及常用快捷键说明。
> 由于EIDE独立安装了JLink，如果Cortex Debug使用的JLink-GDBServer不是EIDE的JLink，因此添加新设备的操作可能需要在其他JLink的安装路径再操作一遍。

## 3.DapLink烧录调试
### 3.1 安装pyOCD
注意：**Python3.12无法安装pyOCD**[^5]。Python3.10.x满足要求。pyOCD安装命令：
```sh
python -m pip install -U pyocd
```
![Python以及pyOCD的版本](py_version.png)
### 3.2 烧录
{% note info %}
使用**pyOCD**烧录固件，理论上兼容所有的DapLink。
{% endnote %}
- 点击**Flasher Configurations**的切换按钮，确保烧录方式为**pyOCD**
- 选择**Target Name**为**hc32f460xe**
- 修改**Download Speed**为**8M**
- 点击**Program Flash**按钮尝试烧录，实测烧录速度比JLink快；同等硬件下也比Keil烧录快。

![DapLink烧录](daplink_flash.gif)

### 3.3 调试
#### 3.3.1 安装OpenOCD
点击**Setup Utility Tools**，选择**OpenOCD**，点击安装
![安装OpenOCD](install_openocd.png)
#### 3.3.2 新增目标配置
增加target配置文件：比如在路径`C:\Users\Orionxer\.eide\tools\openocd_7a1adfbec_mingw32\share\openocd\scripts\target`中，新增**hc32f46x.cfg**文件，**完整复制以下内容并保存**。文件内容来源于羽名大佬[^5]，感谢🎉
{% fold info@hc32f460x.cfg%}
```cfg
source [find target/swj-dp.tcl]
source [find mem_helper.tcl]

# 设置基本环境
if { [info exists CHIPNAME] } {
   set _CHIPNAME $CHIPNAME
} else {
   set _CHIPNAME HC32F460
}

if { [info exists ENDIAN] } {
   set _ENDIAN $ENDIAN
} else {
   set _ENDIAN little
}

if { [info exists WORKAREASIZE] } {
   set _WORKAREASIZE $WORKAREASIZE
} else {
   set _WORKAREASIZE 0x1000
}

if { [info exists CPUTAPID] } {
   set _CPUTAPID $CPUTAPID
} else {
   # set _CPUTAPID 0x2ba01477
   # set _CPUTAPID 0x4ba00477
   set _CPUTAPID 0x2ba01477
}

swj_newdap $_CHIPNAME cpu -irlen 4 -ircapture 0x1 -irmask 0xf -expected-id $_CPUTAPID
dap create $_CHIPNAME.dap -chain-position $_CHIPNAME.cpu
set _TARGETNAME $_CHIPNAME.cpu
target create $_TARGETNAME cortex_m -endian $_ENDIAN -dap $_CHIPNAME.dap

$_TARGETNAME configure -work-area-phys 0x20000000 -work-area-size $_WORKAREASIZE -work-area-backup 0

# 设置flash名
set _FLASHNAME $_CHIPNAME.bank1
# flash bank $_FLASHNAME stm32f1x 0x08000000 0 0 0 $_TARGETNAME

proc test-test { } {
    echo 12345
}

adapter srst delay 100

reset_config srst_nogate

if {![using_hla]} {
    # if srst is not fitted use SYSRESETREQ to
    # perform a soft reset
    debug_level 2
    cortex_m reset_config sysresetreq
    # cortex_m reset_config vectreset
}

$_TARGETNAME configure -event examine-end {
	# DBGMCU_CR |= DBG_WWDG_STOP | DBG_IWDG_STOP |
	#              DBG_STANDBY | DBG_STOP | DBG_SLEEP
   # echo 2222222
	# mmw 0xE0042004 0x00000307 0
}

$_TARGETNAME configure -event trace-config {
	# Set TRACE_IOEN; TRACE_MODE is set to async; when using sync
	# change this value accordingly to configure trace pins
	# assignment
   # echo 333333
	# mmw 0xE0042004 0x00000020 0
}

# mcu不支持软断点. 这时需要openocd启动时设置
#     软断点转为硬件断点, 实际参数有 hard|soft|disable
#     详细可以找相关文档 Server-Configuration.html#gdbbreakpointoverride
gdb_breakpoint_override hard
```
{% endfold %}

![新增目标配置](add_cfg.png)

#### 3.3.3 生成调试配置
{% note warning %}
启动调试前自行确保Cortex Debug配置正确。
{% endnote %}
- EIDE中**右键点击**项目，选择**Generate Debugger Configuration**，选择**OpenOCD**，
- **Interface**选择**cmsis-dap-v1**
- **Target**选择**hc32f46x**
- 一般默认参数不需要修改，点击**Create**
- **F5**启动调试，调试启动较慢，大概耗时10-20秒不等。如果已经存在JLink等其他调试配置，可以先注释以避免无法启动调试。

![生成OpenOCD调试配置](generate_openocd_cfg.png)

![OpenOCD启动调试](openocd_debug.png)

#### 3.3.4 优化调试速度
成功进入调试后，根据gdbserver的log显示，其速度默认在100KHz；找到并修改**cmsis-dap-v1.cfg**文件：指定速度为8000KHz，即8MHz，能够显著提升进入调试的速度，实测启动调试耗时约**4秒**(与Keil的速度一致)，调试过程中没有延时。完整配置与路径如下：
{% fold info@cmsis-dap-v1.cfg%}
```cfg
#
# CMSIS-DAP V1 (CMSIS-DAP Compliant Debugger)
#
adapter driver cmsis-dap
transport select swd
adapter speed 8000
cmsis_dap_backend hid
```
{% endfold %}

![cmsis-dap-v1.cfg路径](cmsis-dap-path.png)

![优化调试速度](openocd_boost.gif)

![调试效果](debug_demo.png)

关于更多FreeRTOS调试分析可以参考STM32-FreeRTOS移植与调试[^7]的**第3点-调试FreeRTOS**和**第4点-监控分析**

{% note warning %}
不选用PyOCD进行调试是因为实测调试速度特别慢，且重启芯片的时候无法再次停在main函数。
{% endnote %}

## 4.参考

[^1]: [Windows搭建STM32开发环境-进阶篇](https://blog.gogo.uno/2024/02/13/vscode-stm32-develop-pro/)
[^2]: [cortex-debug 用法](https://discuss.em-ide.com/blog/67-cortex-debug)
[^3]: [Embedded IDE官方文档](https://em-ide.com/zh-cn/docs/intro/)
[^4]: [Linux搭建C开发环境](https://blog.gogo.uno/2024/02/04/vscode-develop-c/)
[^5]: [Python3.12无法安装pyOCD](https://www.ezurio.com/support/faqs/i-am-having-problems-installing-pyocd-for-python)
[^6]: [羽名/aHC32F460_SDK](https://gitee.com/uyami/a-hc32-f460_-sdk)
[^7]: [STM32-FreeRTOS移植与调试](https://blog.gogo.uno/2024/03/23/stm32-freertos/)