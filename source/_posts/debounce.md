---
title: 按键去抖
date: 2024-03-16 22:50:23
categories:
- STM32
tags: [STM32, C/C++]
banner_img: /images/debounce.jpg
index_img: /images/debounce.jpg
excerpt: 分别对比按键去抖的软硬件实现方案，使用STM32 + 示波器测试去抖效果
---

{% note success %}
由于机械按键的特性（例如轻触开关），在开关的瞬间，不会稳定地闭合或断开，一连串的抖动信号可以轻易被单片机捕获，因此单片机可能会误判按键状态。为了能够让单片机正确识别按键状态，必须对输入的按键信号进行去抖，分为**硬件去抖**和**软件去抖**。实际项目中最常用的去抖方案就是软件**定时器** + 硬件**电容滤波**。
{% endnote %}

## 1.抖动波形
{% note primary %}
正常人类操作按键的时间约几百毫秒到几秒不等，而按键的抖动信号的持续时间约**5ms到20ms**。按键的闭合以及松开的瞬间均会产生抖动信号。
{% endnote %}

![按键抖动波形](bounce.png)

按键在空闲时(稳定松开状态)默认高电平，以下分析**上升沿**：按键在按下状态，松开瞬间的波形，也就是从低电平变为高电平的瞬间。

## 2.创建工程
STM32CubeMX创建工程，设置`PA2`为`GPIO_EXTI2`(配置为外部中断触发)，同时设置`PC13`作为GPIO输出。

![初始化工程](init_project.png)

![配置触发中断条件](config_interrupt.png)

生成工程并导入到`EIDE`中，在`/* USER CODE BEGIN 4 */`代码块之间插入代码，意思是在GPIO的输入中断里切换`PC13`的电平（切换LED灯显示），编译烧录到STM32开发板中
{% fold info@USER CODE BEGIN 4 %}
```c
void HAL_GPIO_EXTI_Callback(uint16_t GPIO_Pin)
{
    // 判断产生中断的引脚
    if (GPIO_Pin == GPIO_PIN_2)
    {
        // 切换LED灯状态
        HAL_GPIO_TogglePin(GPIOC, GPIO_PIN_13);
    }
}
```
{% endfold %}

## 3.波形分析
将一个轻触开关的一端接入`PA2`，另一端接入`GND`，多次按下按键，观察LED状态，发现不能每次都稳定地切换LED灯状态。将示波器接入`PA2`，参数设置如下，开启单次触发，观察波形不难看出在按键松开的过程中发生了多次抖动

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

|触发类型|触发方式|时基|幅值|
| :--:| :--: |:--: | :--:|
|上升沿| 常规|100uS|1V |
</div>

![按键实际抖动波形](osc_bounce.bmp)

## 4.软件去抖
### 4.1 延时
延时的方式相当直观简单，在主循环中，只要检测到`PA2`为低电平，也就是按键按下，**延迟10ms再次检测**是否为低电平，如果是说明按键按下。
{% fold info@ USER CODE BEGIN 3%}
```c
// 如果检测到低电平，即按键按下
if (HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_2) == GPIO_PIN_RESET)
{
    // 延时10ms
    HAL_Delay(10);
    // 再判断一次是否为低电平
    if (HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_2) == GPIO_PIN_RESET)
    {
        // 切换LED显示状态
        HAL_GPIO_TogglePin(GPIOC, GPIO_PIN_13);
        // 等待按键松开
        while (HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_2) == GPIO_PIN_RESET);
    }
}
```
{% endfold %}

![延时判断代码](delay.png)

{% note warning %}
虽然延时判断的方式简单有效，但是会极大影响系统的实时性（系统在延时期间无法处理其他任务）。正常情况下不会应用到实际项目中，除非是简单验证功能或者项目没有实时性的要求。
{% endnote %}

### 4.2 定时器
STM32CubeMX添加定时器`TIM1`，参数如下。定时器中断去抖的本质上就是检测在某一段时间内是否有连续相同的电平状态。
![定时器TIM1配置](tim1.png)
![启用TIM1中断](update_interrupt.png)

以下函数均在`main.c`文件中实现

{% fold info@USER CODE BEGIN PM%}
```c
#define PRESS_STATE 0
#define RELEASE_STATE 1
#define DEBOUNCE_TIME 25
```
{% endfold %}

{% fold info@USER CODE BEGIN PV%}
```c
int old_btn_state = 0;
int new_btn_state = 0;
```
{% endfold %}

{% fold info@USER CODE BEGIN 2%}
```c
HAL_TIM_Base_Start_IT(&htim1);
// 按键状态 初始化
old_btn_state = HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_2);
new_btn_state = HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_2);
```
{% endfold %}

{% fold info@USER CODE BEGIN 3%}
```c
if (old_btn_state != new_btn_state)
{
    // ** 此处可以执行耗时操作
    // 如果按键按下
    if (new_btn_state == PRESS_STATE)
    {
        // TODO VSCode 添加日志点
        HAL_GPIO_TogglePin(GPIOC, GPIO_PIN_13);
    }
    else // 按键松开
    {
        // TODO VSCode 添加日志点
    }
    // 更新按键状态
    old_btn_state = new_btn_state;
}
```
{% endfold %}

{% fold info@USER CODE BEGIN 4%}
```c
void button_scan(int *state)
{
    // 按下时长
    static int press_time = 0;
    // 松开时长
    static int release_time = 0;
    // 如果读取到低电平
    if (HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_2) == GPIO_PIN_RESET)
    {
        // 清零松开时长
        release_time = 0;
        // 累计按下时长
        press_time++;
        // 如果定时器检测到连续25ms都是低电平，说明按键按下状态有效
        if (press_time >= DEBOUNCE_TIME)
        {
            *state = PRESS_STATE;
        }
    }
    else // 如果读取到高电平
    {
        // 清零按下时长
        press_time = 0;
        // 累计松开时长
        release_time++;
        // 如果定时器检测到连续25ms都是高电平，说明按键松开状态有效
        if (release_time >= DEBOUNCE_TIME)
        {
            *state = RELEASE_STATE;
        }
    }
}

void HAL_TIM_PeriodElapsedCallback(TIM_HandleTypeDef *htim)
{
    if (htim == &htim1)
    {
        // 判断并更新按键状态
        button_scan(&new_btn_state);
    }
}
```
{% endfold %}

## 5.硬件去抖
### 5.1 电容滤波
{% note primary %}
最常见的硬件去抖方式是电容滤波，给按键并联一个电容和串联一个电阻，利用电容的充放电特性实现一定程度的滤波。
{% endnote %}

以电容取值100nF，电阻取值10KΩ为例，原理图如下，将示波器探头接入`PA2`，观察按键松开时的波形

![电容滤波原理图](schematic.png)

![电容滤波结果](capacitance.bmp)

底部的测量参数**上升沿**1.524ms，实际从低电平爬升至高电平需要3-4ms左右，因此电容的取值不能过大，否则爬升时间太长可能会影响按键的有效判断。

### 5.2 RS触发器
{% note primary %}
RS触发器就是使用两个**与非门**的级联，其中一个与非门的输出作为另一个与非门的输入，描述方法是现态$Q^{n}$和次态$Q^{n+1}$，通常用于没有MCU的场合，使用较少。
{% endnote %}

单击此处可以[在线模拟RS触发器](http://scratch.trtos.com/circuitjs.html?cct=$+1+0.000005+20.712724888983452+54+5+43%0A151+160+80+416+80+0+2+5+5%0A151+160+256+416+256+0+2+5+5%0Ar+416+80+496+80+0+1000%0Ag+496+80+544+80+0%0Aw+112+304+112+320+0%0Av+112+448+112+384+0+0+40+5+0+0+0.5%0Ag+112+448+112+496+0%0AS+-208+176+-112+176+0+0+false+0+2%0Ag+-208+176+-240+176+0%0Ar+112+320+112+384+0+1000%0Aw+112+304+112+272+0%0Aw+-48+272+-48+192+0%0Aw+-48+192+-112+192+0%0Ar+112+-32+112+32+0+1000%0Av+112+-112+112+-32+0+0+40+5+0+0+0.5%0Ag+112+-112+112+-144+0%0Aw+112+272+-48+272+0%0Aw+112+32+112+64+0%0Aw+112+64+-48+64+0%0Aw+-48+64+-48+160+0%0Aw+-48+160+-112+160+0%0Aw+160+64+112+64+0%0Aw+160+272+112+272+0%0Aw+160+96+160+144+0%0Aw+160+240+160+208+0%0Aw+160+144+416+208+0%0Aw+416+208+416+256+0%0Aw+160+208+400+128+0%0Aw+400+128+400+80+0%0Ao+2+64+0+4098+5+0.05+0+2+2+3%0A)按键效果，单击切换开关，观察底部的示波器波形，示意图如下

![模拟RS触发器](rs_schematic.png)

根据与非门的先与后非的运算规则，**只要输入0，一定输出1**，真值表如下
<div class="center">

|R输入|S输入|Q输出|
| :--:|:--:|:--:|
|  0  |  0 |  1 |
|  0  |  1 |  1 |
|  1  |  0 |  1 |
|  1  |  1 |  0 |
</div>

- *当R1稳定连接*，`R1 = 0`，则S2一定稳定断开(单刀双掷开关不能可能同时导通), `S2 = 1`；根据与非门规则：`Q1 = 1`；根据图中R2与Q1直连，因此`R2 = Q1 = 1`, 因此`Q2 = 0`；S1与Q2直连，因此`S1 = Q2 = 0`
- *当R1开始断开*, R1状态不稳定，随机1或0，但是因为且S2状态不变，`S1 = 0`也不变，因此Q1状态不变`Q1 = 1`
- *当R1完全断开*，`R1 = 1`，Q1保持
- *当S2开始连接*，S2状态不稳定，随机1或0；当S2第一次等于0，则`Q2 = 1`，`S1 = Q2 = 1`，因为R1稳定断开`R1 = 1`，因此`Q1 = 0`，`R2 = Q1 = 0`，后续就算是S2再变成1，因为R2已经稳定，因此Q1稳定
- *当S2稳定连接*，`S2 = 0`，Q1不变`Q1 = 0`

{% note info %}
按照目前按键的生产工艺，抖动的情况已经有比较好的改善，甚至有些品质较好的按键都已经不会产生抖动信号。
另外，有一些单片机在其GPIO功能上就内置了按键去抖功能，在初始化GPIO输入的时候，设置结构体的`deboune`成员变量为**10ms**即可。相对与自行实现的按键去抖方案，不仅实现起来简单，而且还节省一个定时器资源。
{% endnote %}

## 参考
[^1]: [HAL库 STM32CubeMX----外部中断与按键中断](https://blog.csdn.net/weixin_62213694/article/details/124557870)

[^1]: [Timer 与按键消抖的应用](https://c.miaowlabs.com/A22.html)
