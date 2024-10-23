---
title: STM32-串口USART
date: 2024-03-02 20:16:39
categories:
- STM32
tags: [STM32, C/C++]
banner_img: /images/stm32-uart.jpg
index_img: /images/stm32-uart.jpg
excerpt: 使用STM32CubeMX+STM32配置UART外设以及设置为系统调试打印的输出接口
---

{% note success %}
本篇文章介绍STM32CubeMX+STM32配置UART外设生成MDK工程，分别使用串口调试工具、示波器和逻辑分析仪分析串口数据；设置USART1为系统调试打印的输出接口
{% endnote %}

## 1.准备
- USB转TTL串口模块，CH34X系列或CP210X系列或者FT232系列等任意USB转串口模块均可
- 串口监视软件，最好支持ANSI转义序列以及UTF-8编码，比如SerialPortForWindowsTerminal[^1]或者MobaXterm
- 璞石示波器 或 其他示波器，最好可以支持协议解码
- 逻辑分析仪，使用Saleae Logic 2软件，硬件自行选择
- USB转TTL串口模块与STM32接线，**RX接对端TX**，反之同理，接线方式如下

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

|      STM32引脚      |     USB转串口模块   |    
| :----------------:  | :-----------------:|
| <font color=Red>USART1_RX即PA9 </font>   |   <font color=Red>TX 或 TXD</font>|
| <font color=Green>USART1_TX即PA10</font>  |  <font color=Green>RX 或 RXD</font> |
|         GND         |         GND        |
</div>

## 2.生成MDK工程
使用STM32CubeMX生成Keil MDK工程，目标芯片选择**STM32F103C8**，时钟配置为**72MHz**，步骤基本与[Windows搭建STM32开发环境(基础篇)](https://blog.gogo.uno/2024/02/12/vscode-stm32-develop/)一致
- 配置**PC13**为**GPIO输出**
- 配置串口外设**USART1**，选择模式为`Asynchronous`，则**PA10**和**PA9**默认为USART1的RX与TX

![增加USART1配置](add_usart1.png)

- 点击`NVIC Settings`选项卡，勾选**USART1 global interrupt**，也就是开启USART1的全局中断

![开启串口接收中断](enable_interrupt.png)

## 3.编译烧录
在工程生成之后，默认使用Keil打开工程，设置Jlink烧录，选择SWD方式，勾选`Reset and Run`即下载完自动重启，在`main.c`文件中的`/* USER CODE BEGIN 3 */`后添加代码
```c
// LED切换高低电平
HAL_GPIO_TogglePin(GPIOC, GPIO_PIN_13);
char *hello_str = "hello world\n";
// USART1 发送字符串
HAL_UART_Transmit(&huart1, (uint8_t *)hello_str, strlen(hello_str), HAL_MAX_DELAY);
HAL_Delay(500);
```
尝试编译烧录，如果下载成功PC13的LED灯会以500ms的间隔闪烁，打开串口调试助手，也能看到STM32的串口以500ms的间隔发送字符串

## 4.EIDE导入工程
VSCode打开EIDE插件，选择`import_project`，在弹出的窗口中选择MDK工程，确定切换工作区，选择烧录的芯片为`STM32F103C8`，尝试编译烧录
![EIDE编译STM32工程](eide_compile.png)

## 5.串口接收中断
{% note primary %}
STM32接收串口的数据方式之一是**打开**串口的接收中断`HAL_UART_Receive_IT`，也可以叫**使能**或者**启用**串口的接收中断，本质上都是**Enable**。以下为一个例子：在产生串口中断后，将串口的接收数据拷贝到接收缓冲区中，等待接收结束标志位的出现(即换行符)，则认为单次串口接收结束，置位接收标志位，通知主函数打印接收到的所有数据，并重新打开接收中断。
{% endnote %}

### 5.1 代码实现
以下代码均在`main.c`文件中编辑，复制以下代码到STM32CubeMX生成代码的特殊**注释**处,尝试编译下载
{% fold info@接收缓冲宏 %}
```c
/* Private define ------------------------------------------------------------*/
/* USER CODE BEGIN PD */
// 串口接收缓冲区大小
#define RX_MAX_LEN 128 
// 串口接收结束标志，即换行符
#define LINE_FEED       '\n'
/* USER CODE END PD */
```
{% endfold %}
{% fold info@接收缓冲区变量 %}
```c
/* Private variables ---------------------------------------------------------*/
/* USER CODE BEGIN PV */
// 串口中断接收字节
uint8_t rx_byte = 0;
// 串口接收缓冲区
uint8_t rx_buf[RX_MAX_LEN] = {0};
// 串口接收完成标志
uint8_t rx_flag = 0;
// 串口接收实际长度
uint8_t rx_cnt = 0;
/* USER CODE END PV */
```
{% endfold %}
{% fold info@开启中断 %}
```c
/* USER CODE BEGIN 2 */
char *hello_str = "STM32 USART1 Demo\n";
// USART1 发送字符串
HAL_UART_Transmit(&huart1, (uint8_t *)hello_str, strlen(hello_str), HAL_MAX_DELAY);
// 打开串口接收中断
HAL_UART_Receive_IT(&huart1, &rx_byte, 1);
/* USER CODE END 2 */
```
{% endfold %}
{% fold info@主循环处理 %}
```c
/* Infinite loop */
/* USER CODE BEGIN WHILE */
while (1)
{
    /* USER CODE END WHILE */
    /* USER CODE BEGIN 3 */
    if (rx_flag)
    {
        // 切换LED灯
        HAL_GPIO_TogglePin(GPIOC, GPIO_PIN_13);
        // 打印接收到的数据
        HAL_UART_Transmit(&huart1, rx_buf, rx_cnt, HAL_MAX_DELAY);
        rx_flag = 0;
        rx_cnt = 0;
        memset(rx_buf, 0, sizeof(rx_buf));
        // 打开接收中断
        HAL_UART_Receive_IT(&huart1, &rx_byte, 1);
    }
    
}
/* USER CODE END 3 */
```
{% endfold %}
{% fold info@接收中断%}
```c
/* USER CODE BEGIN 4 */
void HAL_UART_RxCpltCallback(UART_HandleTypeDef *huart)
{
    if (huart->Instance == USART1)
    {
        rx_buf[rx_cnt++] = rx_byte;
        if (rx_byte == LINE_FEED)
        {
            rx_flag = 1; // 标记接收结束
        }
        else
        {
            // 继续打开接收中断
            HAL_UART_Receive_IT(huart, &rx_byte, 1);
        }
    }
}
/* USER CODE END 4 */
```
{% endfold %}
### 5.2 串口调试
打开友善串口调试助手[^2],根据实际情况选择有效的串口端口，串口参数与STM32CubeMX配置的USART1一致，如下方所示

<div class="center">

|波特率|数据位|校验位|停止位|流控| 
| :--:| :--: | :--:| :--: |:--:|
|115200| 8   |None |  1   |None|
</div>

编译烧录到STM32，尝试发送一个字符串`Echo test`，字符串结尾加一个回车符标记串口发送结束，观察串口助手是否将收到的数据发送回来。

![串口收发](uart_transceive.gif)

{% note warning %}
如果尝试给STM32的串口发送一个字符串，且确定加了换行作为结束标志，但是STM32没有将数据发送回来，那么应该需要检查串口调试的助手的**换行设置**, 确保是**Line Feed**而不是*Carriage Return*。按照当前代码的结束符，如果不是以**Line Feed**结尾则无法认为本次接收结束
{% endnote %}

<div class="center">

|换行符|缩写|ASCII码|16进制|
| :--:| :--: |:--: | :--:|
|Carriage Return| CR|\r   |0x0D |
|Line Feed| LF  |\n   |0x0A |
</div>

![换行设置为Line Feed](return.png)

## 6.示波器
{% note primary %}
使用示波器查看数据，重点是测量信号、波形分析。
{% endnote %}
### 6.1 参数设置
硬件连接：通道0连接串口模块的TX，通道1连接RX。
串口在空闲时保持高电平，一个串口数据帧 = 1bit起始位 + 8bit数据位 + 1bit停止位。起始位低电平，停止位高电平。因此示波器触发类型需要选择**下降沿**，并启用**单次触发**，具体参数见下表
<div class="center">

|触发类型|触发方式|时基|幅值|
| :--:| :--: |:--: | :--:|
|下降沿| 常规|200uS|2V |
</div>

### 6.2 捕获波形
- 示波器在设置好参数之后开始捕获波形，等待串口的开始信号触发，也就是第一个下降沿开始采样。
- 串口调试助手发送一条数据`Echo Test`
- 查看示波器波形，能够看到通道0先采集到数据，通道1再采集到数据

![捕获波形](osc_capture.bmp)

### 6.3 波形分析
选择示波器的**UART解码**功能，暂时分析通道0，串口参数与上面保持一致，注意位序默认是**LSB**
![串口解码](osc_uart_decode.bmp)

放大最左侧的起始信号，分析第一个字节`0x45`
- `S`代表起始位，低电平
- 中间代表数据位，共8位，从高到低的二进制是`1010 0010`，由于串口传输的方式默认是**LSB**，因此该二进制需要反转才能代表正确数据，即`0100 0101`，换算成16进制也就是`0x45`，也就是ASCII码的字符`E`
- `T`代表停止位，高电平

![分析第一个字节](decode_first_byte.bmp)

### 6.4 字符解码
设置串口解码的数据显示类型为ASCII字符，且放大时基，查看通道1所有数据

![解码通道0的ASCII码](decode_ascii_channel_1.bmp)

![解码通道1的ASCII码](decode_ascii_channel_2.bmp)

## 7.逻辑分析仪
{% note primary %}
使用逻辑分析仪查看数据，重点是解码协议。虽然逻辑分析仪在测量信号的准确度上不如示波器，只能处理逻辑电平信号：高电平`1`或低电平`0`，但其在解码协议以及数据显示方面是十分优秀的。
{% endnote %}
### 7.1 参数设置
硬件连接：`Channel 0`连接串口模块的TX，`Channel 1`连接RX
- 点击`Analyzers`标签
- 选择`Async Serial`，选择`Channel 0`，串口其他参数保持与上面一致，点击`Save`
- 同理添加`Channel 1`
- 重命名`Channel 0`为`UART_TX`， `Channel 0`为`UART_RX`，并选择合适的颜色

![设置串口通道参数](set_logic.gif)

### 7.2 监听数据
- 点击`Start`开始监听，并适当放大时基
- 串口调试助手发送一条数据`Echo Test`
- 停止监听，缩小时基，查看具体数据

![监听数据](logic_capture.gif)

### 7.3 分析数据
选择`Channel 0`的数据显示方式为`Hexadecimal`，以16进制查看数据；放大第一个字节，切换不同数据显示格式以验证以上结果。

![分析数据](logic_analyze.gif)

{% note warning %}
**示波器查看信号是否准确，逻辑分析仪查看数据是否正确。**
{% endnote %}

## 8.重定向printf
{% note primary %}
重定向`printf`就是将串口的发送函数转换为标准的`printf`函数以方便打印调试信息。并且可以导入彩色打印库[^4]以打印显示友好的调试信息。
{% endnote %}

### 8.1 启用微库
- EIDE工程选择`Builder Configurations`， 选择编辑`Builder Options`
- 弹出的`Build Options`页面中选择`Global Options`选项卡
- 确保勾选**Use MicroLIB**，点击右上角`SAVE ALL`按钮保存

![启用微库](use_microlib.png)

### 8.2 重定向设置
#### 8.2.1 修改头文件
在`usart.h`文件中，在 `/* USER CODE BEGIN Includes */` 与` /*USER CODE END Includes */`之间加入` #include <stdio.h>`

![usart.h导入stdio.h](uasrt.h.png)

#### 8.2.2 修改源文件
在`usart.c`文件中，在 `/* USER CODE BEGIN 0 */` 和 `/* USER CODE END 0 */`加入以下代码
{% fold info@USER CODE BEGIN 0 %}
```c
#ifdef __GNUC__
/* With GCC/RAISONANCE, small printf (option LD Linker->Libraries->Small printf
set to 'Yes') calls __io_putchar() */
#define PUTCHAR_PROTOTYPE int __io_putchar(int ch)
#else
#define PUTCHAR_PROTOTYPE int fputc(int ch, FILE *f)
#endif /* __GNUC__ */
```
{% endfold %}

![增加代码到usart.c的USER CODE 0](usart.c_1.png)

在`usart.c`文件中，在 `/* USER CODE BEGIN 1 */` 和 `/* USER CODE END 1 */`加入以下代码
{% fold info@USER CODE BEGIN 1 %}
```c
/**
* @brief  Retargets the C library printf function to the USART.
* @param  None
* @retval None
*/
PUTCHAR_PROTOTYPE
{
    /* Place your implementation of fputc here */
    /* e.g. write a character to the EVAL_COM1 and Loop until the end of transmission */
    HAL_UART_Transmit(&huart1, (uint8_t *)&ch, 1, HAL_MAX_DELAY);
    return ch;
}
```
{% endfold %}

![增加代码到usart.c的USER CODE 1](usart.c_2.png)

### 8.3 标准打印
在`main.c`文件中，尝试在串口初始化完成后使用`printf`函数打印`STM32 USART1 Demo`字符串

![标准打印](stanrd_print.png)

### 8.4 彩色打印
#### 8.4.1 文件准备
- MDK同级目录下**新建**`Debug`目录
- 将彩色打印库[^4]，即`debug_log.h`和`debug_log.c`文件复制在`Debug`目录之下

![文件夹准备](new_debug_folder.gif)

#### 8.4.2 添加至工程
- 点击`Project Resources`的右侧文件夹按钮，选择`Virtual Folder`，输入**Debug**
- 选择`debug_log.c`文件
- 点击`Project Attributes`，选择`Include Directories`右侧的加号，选择`Debug`目录

![添加至工程](add_debug_log.gif)


#### 8.4.3 打印效果
- 导入头文件`debug_log.h`
- 在进入循环之前调用`debug_log_demo()`
- 编译烧录，查看打印效果

![彩色打印效果](log_demo.gif)


{% note warning %}
注意：`printf`函数不能在硬件级的中断函数中打印，该函数阻塞运行且属于耗时操作。比如`printf`打印一个字节(有效数据8位)，以波特率115200，1bit起始位 + 8bit数据位 + 1bit停止位 为例，传输一个字节大约需要86us。假设一个定时器频率1Mhz, 其中断函数需要1us进入一次，`printf`的执行时间远大于该定时器进入中断的时间。传输一个字节的时间计算公式如下:
{% endnote %}

$$
一个字节传输时间 = (1bit + 8bit + 1bit) ÷ 115200bps ≈ 0.000086s = 86us 
$$

![测量传输一个字节的时长](1byte_time.gif)


## 参考
[^1]: [SerialPortForWindowsTerminal](https://github.com/Zhou-zhi-peng/SerialPortForWindowsTerminal?tab=readme-ov-file)
[^2]: [友善串口调试助手](https://www.alithon.com/downloads)
[^3]: [Usart与Printf函数重定向](https://c.miaowlabs.com/A23.html)
[^4]: [终端彩色打印](https://blog.gogo.uno/2024/02/12/c-color-print/)

