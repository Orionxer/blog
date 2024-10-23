---
title: VSCode搭建ESP32开发环境 
date: 2024-01-27 02:38:30
categories:
- ESP32
tags: [VSCode, ESP32]
banner_img: /images/vscode-esp32-develop.png
index_img: /images/vscode-esp32-develop.png
excerpt: 在Win10下搭建VSCode+ESP32开发调试环境，实现串口以及JTAG烧录调试，包含一键自动编译烧录调试，实现优雅地开发ESP32。
---

{% note success %}
在Win10下搭建VSCode+ESP32开发调试环境，实现串口以及JTAG烧录调试，包含一键自动编译烧录调试，实现优雅地开发ESP32。本篇文章使用**ESP32-S3**模组进行测试，因为S3自带USB转JTAG调试功能；**如果只使用串口烧录调试则所有ESP系列芯片通用**。
{% endnote %}

## 1.硬件准备
- `ESP32-S3-WROOM`开发板(双Type-C接口：一个USB转串口，一个USB转JTAG)
- 示波器，测量引脚输出波形

![ESP32-S3-WROOM开发板](s3_dev.png)

## 2.VSCode安装ESP-IDF插件
### 2.1 根据ESP-IDF插件提示安装环境
完全按照ESP-IDF的提示进行安装即可，基本上全自动，**全程开启代理确保下载速度最优**
- 选择侧边栏`ESP-IDF`选项卡
- 选择`Configure ESP-IDF Extension`
- 选择`EXPRESS`(也可以根据个人实际情况选择安装模式)
- 选择下载的服务器以及`ESP-IDF`的版本，一般默认选择最新的，安装路径需要注意：将所有`Administrator`替换为自己的英文用户名，比如`Orionxer`，**尽量避免权限问题**
- 慢慢等待安装结束，大约需要30分钟左右，VSCode会提示下载安装过程

![选择安装模式](setup_mode.png)
![选择下载服务器以及ESP-IDF的版本](install.png)

### 2.2 新建项目
- ESP-IDF选项卡侧边栏选择`Show Examples`
- 选择默认的`ESP-IDF`环境
- 创建`blink`项目,选择项目文件夹以保存

![新建项目](new_project.gif)

### 2.3 选择目标芯片
点击底部栏的`Set Espressif Target(IDF_TARGET)`选择目标芯片
- 选择工程`blink`
- 选择`esps3`
- 选择`ESP32-S3 Chip (via built-in USB-JTAG)`

![选择目标芯片](select_target.gif)

### 2.4 编译项目
点击底部栏的编译按钮，第一次编译的时间会稍长，后续如果修改了项目的`Configuration`(即menuconfig)也会导致编译时间变长。编译成功后会输出RAM/ROM信息。

![编译项目](compile.png)

{% note warning %}
如果VSCode在`c_cpp_properties.json`文件中提示`Cannot find "${env:IDF_TOOLS_PATH}`等类似错误，
找到`compilerPath`键值对，将`config:idf.toolsPath`修改为`config:idf.toolsPathWin`即可
{% endnote%}

## 3.串口
#### 3.1 烧录
使用串口烧录前需要将丝印`COM`接入电脑，正常情况下Win10会自动安装驱动，并识别出正确的端口、选择该端口，比如`COM8`，点击烧录。

![串口烧录](uart_flash_default.gif)

#### 3.2 监控
点击监控按钮，`ESP-IDF`启动串口监控命令行界面，可以观察到每秒打印一个日志。快捷键`Ctrl + ]`退出串口监控。

![串口监控](uart_monitor.gif)

#### 3.3 一键编译烧录监控
`ESP-IDF`针对串口方式，提供了一键编译烧录监控的按钮，方便快速调试验证。

![串口一键编译烧录监控](uart_build_flash_monitor.gif)


#### 3.4 提升烧录速度
在烧录过程中，如果不指定波特率，则烧录速度会默认为`460800bps`。可以通过`settings.json`文件修改键值对`"idf.flashBaudRate"`的值，提升烧录速度，最高速度取决于USB转串口芯片最大支持的波特率。

##### 3.4.1 ESP32-S3
以`ESP32-S3-WROOM`开发板为例，板载**CH343P**USB转串口芯片，根据该芯片的数据手册，理论波特率最高能设置为`6Mbps`[^1]，修改为`4000000`即`4Mbps`可以烧录。
```json
"idf.flashBaudRate": "4000000"
```

{% fold info @settings.json示例 %}
```json
{
  "C_Cpp.intelliSenseEngine": "default",
  "idf.adapterTargetName": "esp32s3",
  "idf.openOcdConfigs": [
    "board/esp32s3-builtin.cfg"
  ],
  "idf.flashType": "UART",
  "idf.portWin": "COM8",
  "idf.flashBaudRate": "4000000",
  "files.associations": {
    "task.h": "c",
    "freertos.h": "c",
    "freertosconfig.h": "c"
  }
}
```
{% endfold %}

{% note warning %}
实测修改为`6000000`即`6Mbps`，无法烧录，待验证...
{% endnote %}

##### 3.4.2 ESP32
以`ESP-WROOM-32`开发板为例，板载**CP2102**的USB转串口芯片，根据该芯片的数据手册，理论波特率最高能设置为`1Mbps`[^2]。实测修改为`1000000`即`1Mbps`，无法烧录，只是`921600bps`相当接近于`1Mbps`。
```json
"idf.flashBaudRate": "921600"
```
保存并尝试烧录，可以看到终端提示烧录的速度是`921600`
![修改波特率](change_baudrate.png)


## 4.JTAG
`ESP32-S3`模组**自带USB转JTAG**功能，配合`OpenOCD`能够实现烧录调试功能。调试功能一般用于不方便串口打印的场景，比如中断函数；另外定位`HardFault`原因比如指针越界访问等问题也会更加精准。

### 4.1 更换驱动
将丝印**USB**的TypeC接口接入电脑。

#### 4.1.1 下载Zadig
进入[Zadig](https://zadig.akeo.ie/)软件官网下载，安装并打开。

![下载Zadig](download_zadig.png)

#### 4.1.2 选择接口
- 点击`Options`，选择`List All Devices`
- 选择`USB JTAG/serial debug unit(Interface 0)`接口，注意一定要选择**Interface 0**接口
- 目的驱动选择`USB Serial CDC`
- 点击Replace Driver

![选择接口更换驱动](replace_driver.png)

更换驱动成功后，重新插拔TypeC接口。设备管理器界面中，`Interface 0`出现在端口(COM和LPT)处，说明驱动更换成功。

![驱动识别成功](replace_driver_success.png)

{% note info %}
该驱动更换仅针对特定的USB端口，如果下次接入的其他的USB端口，则需要再次更换驱动。
{% endnote %}

### 4.2 修改优化等级
进入调试前，确保编译项目的**优化等级为-OG**，否则调试容易出现无法打断点以及断点位置偏移等问题。
- 点击底部栏配置按钮(menuconfig)
- 输入`optimization`，将`Size (-Os)`修改为`Size (-OG)`
- 点击`Save`，重新编译

![修改优化等级](optimization.gif)

### 4.3 烧录
- 点击底部栏的烧录方式，选择`JTAG`
- 点击启动`OpenOCD`。如果`OpenOCD`启动失败可以尝试重启电脑。
- 点击烧录，等待提示烧录成功。(启动`OpenOCD`等待几秒之后再开始烧录)

![JTAG烧录](jtag_flash.gif)

{% note primary %}
修改代码后，进入调试前必须**编译**并且**下载**，否则调试容易出现异常。
{% endnote %}

### 4.4 启动调试

{% fold info @适当修改blink_example_main.c以方便调试%}
```c
void app_main(void)
{
    /* Configure the peripheral according to the LED type */
    configure_led();
    while (1) {
        // ESP_LOGI(TAG, "Turning the LED %s!", s_led_state == true ? "ON" : "OFF");
        blink_led();
        /* Toggle the LED state */
        s_led_state = !s_led_state;
        // vTaskDelay(CONFIG_BLINK_PERIOD / portTICK_PERIOD_MS);
        vTaskDelay(1);
    }
}
```
{% endfold %}

重新编译烧录，在`blink_example_main.c`文件下按下`F5`尝试启动调试。一般VSCode会自动生成`luanch.json`文件配置，如果没有则复制以下的完整示例。

![JTAG调试](jtag_debug.gif)

{% fold info @luanch.json示例 %}
```json
{
    "configurations": [
        {
            "name": "ESP-IDF Debug: Launch",
            "type": "espidf",
            "request": "launch"
        }
    ]
}
```
{% endfold %}


{% fold info @settings.json示例 %}
```json
{
  "C_Cpp.intelliSenseEngine": "default",
  "idf.adapterTargetName": "esp32s3",
  "idf.openOcdConfigs": [
    "board/esp32s3-builtin.cfg"
  ],
  "idf.flashType": "JTAG",
  "idf.portWin": "COM8",
  "idf.flashBaudRate": "4000000",
  "files.associations": {
    "task.h": "c",
    "freertos.h": "c",
    "freertosconfig.h": "c"
  }
}
```
{% endfold %}

{% note warning %}
实测JTAG调试经常启动失败，失败报错原因：`Debug adapter -> Extension: DEBUG_ADAPTER_READY2CONNECT`。如果调试启动失败则多试几次，直至成功为止。暂未找到解决问题的办法。
{% endnote %}

### 4.5 波形测量
#### 4.5.1 100Hz滴答
`blink`项目默认配置了`GPIO48`为GPIO输出引脚，且FreeRTOS的滴答频率是100Hz，即中断时间是10ms。修改`Blink LED type`为`GPIO`，编译烧录，示波器测量`GPIO48`引脚，波形如下：

![100Hz滴答-翻转电平](100hz.png)

根据上述代码分析，`vTaskDelay(1);`代表延迟一个滴答时间。理论上`GPIO48`在延迟10ms后会翻转电平。通过测量**正脉宽**参数可以验证该结论。

#### 4.5.1 1KHz滴答
在不修改以上测试代码的前提下，修改项目配置的滴答频率为`1000Hz`，重新编译烧录，再次测量该引脚，此时**正脉宽**应该是1ms。此时`vTaskDelay(1);`依然代表延迟一个滴答时间，只是该滴答时间为1ms。

![设置FreeRTOS的滴答频率为1KHz](set_1khz.png)

![1KHz滴答-翻转电平](1Khz.png)

## 5.快捷键
合理使用快捷键能够有效提高效率，ESP-IDF的快捷键有点特殊，比如根据ESP-IDf的插件说明：编译项目快捷键是`Ctrl E B`，则按住`Ctrl + E`，**松开**，再按`B`，才能正确触发快捷键

![快捷键](shortcut.png)

常用的快捷键参考
<style>
.center 
{
  width: auto;
  display: table;
  margin-left: auto;
  margin-right: auto;
}
</style>
<div class="center">

| 快捷键 |   英文    | 中文 |
| :----: | :------: | :--: |
|  `Ctrl + E + B`  | Build your project | 编译  |
|  `Ctrl + E + F`  | Flash your project | 烧录  |
|  `Ctrl + E + M`  | Monitor your device | 监控  |
|  `Ctrl + E + D`  | Build, Flash and start a monitor on your device | 编译烧录监控  |
</div>

## 6.参考链接

[^1]: [Ch343: 支持通讯波特率50bps～6Mbps](https://www.wch.cn/products/CH343.html)
[^2]: [CP2102: Baud rates: 300 bps to 1 Mbps](https://www.silabs.com/documents/public/data-sheets/CP2102-9.pdf)
[^3]: [配置 ESP-WROVER-KIT 上的 JTAG 接口](https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32/api-guides/jtag-debugging/configure-ft2232h-jtag.html)