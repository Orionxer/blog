---
title: 终端彩色打印
date: 2024-02-12 17:59:27
categories:
- C/C++
tags: [C/C++, ANSI转义序列]
banner_img: /images/c-color-print.jpg
index_img: /images/c-color-print.jpg
excerpt: 通过ANSI转义序列，使用C语言实现终端彩色打印
---

{% note success %}
通过`ANSI`转义序列，C语言工程中导入一个头文件和一个源文件就可以实现终端彩色打印，并且可以自定义需要显示的打印等级。同时能够以16进制格式化打印数组内容。该彩色打印完全兼容`printf`函数。
{% endnote %}

## 1.要求
要求终端支持`ANSI`转义序列的显示，~~否则无法显示正确的颜色且会打印多余的字符。~~ **最新工程可以关闭颜色输出**，设置宏`COLOR_ENABLE`为`0`以关闭颜色输出，在一些不支持ANSI显示的终端可以设置。另外，如果需要支持中文显示，则终端需要支持`UTF-8`。比如VSCode内置的终端、WSL2的终端、Git Bash等均支持`ANSI`转义序列和`UTF-8`

## 2.入门使用
### 2.1 导入库
可以通过以下提供的头文件和源文件复制粘贴到指定的工程目录，也可以通过github下载[^1]。`debug_log.c`提供了一个示例函数`debug_log_demo`，直接调用观察输出效果。

{% fold info@debug_log.h %}
```c
#ifndef DEBUG_LOG_H_
#define DEBUG_LOG_H_
#include <stdint.h>
#include <string.h>

/**
 * @brief   尽可能确保显示的终端支持ANSI的转移序列以及UTF-8的显示
 * @note    当终端不支持ANSI的转移序列，则无法正常显示颜色信息，
 *          可以通过设置宏COLOR_ENABLE为0关闭颜色输出
 * @note    当终端不支持UTF-8，则关于中文的显示会乱码
*/

/************************** 用户自定义宏 *********************************/
/**
 * @brief   调试输出总开关
 * @note    处于禁用状态时关闭所有输出
*/
#define DBG_ENABLE          1

/**
 * @brief   是否启用颜色输出
 * @note    如果显示的终端不支持ANSI转义序列，可以将此项设置为0以关闭颜色输出
*/
#define COLOR_ENABLE        1

/**
 * @brief   指定 输出打印等级
 * @note    1. 只会打印当前等级及其以上的等级的信息
 * @note    2. 假设 指定的打印等级 = DBG_LOG_WARNING,
 *          则只会打印 DBG_LOG_WARNING 和 DBG_LOG_ERROR两个等级的信息
*/
#define DBG_LOG_LEVEL       DBG_LOG_DEBUG


/*************************** 调试输出保留宏 ********************************/
/**
 * @brief   输出的等级从高到低的排序: [错误] > [警告] > [普通] > [调试]
*/
// [错误]输出等级 
#define DBG_LOG_ERROR       1
// [警告]输出等级
#define DBG_LOG_WARNING     2
// [普通]输出等级
#define DBG_LOG_INFO        3
// [调试]输出等级
#define DBG_LOG_DEBUG       4

#define ANSI_COLOR_RED      "\x1b[31m"
#define ANSI_COLOR_GREEN    "\x1b[32m"
#define ANSI_COLOR_YELLOW   "\x1b[33m"
#define ANSI_COLOR_BLUE     "\x1b[34m"
#define ANSI_COLOR_MAGENTA  "\x1b[35m"
#define ANSI_COLOR_CYAN     "\x1b[36m"
#define ANSI_COLOR_RESET    "\x1b[0m"

#if COLOR_ENABLE
#define PRINT_ANSI_COLOR(...) printf(__VA_ARGS__)
#else
#define PRINT_ANSI_COLOR(...)
#endif


// 调试输出总开关 处于打开状态
#if DBG_ENABLE
// 日志打印格式
#define DBG_LOG(color, ...)                 \
        PRINT_ANSI_COLOR(color);            \
        printf("[%s]: ", __func__);         \
        printf(__VA_ARGS__);                \
        PRINT_ANSI_COLOR(ANSI_COLOR_RESET); \
        printf("\n");                       

// [调试]等级控制
#if DBG_LOG_LEVEL >= DBG_LOG_DEBUG
#define DBG_LOGD(...) DBG_LOG(ANSI_COLOR_BLUE, __VA_ARGS__);
#else
#define DBG_LOGD(...)
#endif

// [普通]等级控制
#if DBG_LOG_LEVEL >= DBG_LOG_INFO
#define DBG_LOGI(...) DBG_LOG(ANSI_COLOR_GREEN, __VA_ARGS__);
#else
#define DBG_LOGI(...)
#endif

// [警告]等级控制
#if DBG_LOG_LEVEL >= DBG_LOG_WARNING
#define DBG_LOGW(...) DBG_LOG(ANSI_COLOR_YELLOW, __VA_ARGS__);
#else
#define DBG_LOGW(...)
#endif

// [错误]等级控制
#if DBG_LOG_LEVEL >= DBG_LOG_ERROR
#define DBG_LOGE(...) DBG_LOG(ANSI_COLOR_RED, __VA_ARGS__);
#else
#define DBG_LOGE(...)
#endif
#else
#define DBG_LOGD(...)
#define DBG_LOGI(...)
#define DBG_LOGW(...)
#define DBG_LOGE(...)
#endif

/**
 * @brief   自定义打印
 * @note    不受级别控制，用于打印系统开机提示等关键信息
 * @note    第一个参数传入颜色，第二个参数传入需要打印的信息
 * @note    最好传入两个宏以避免警告
*/
#define ADVANCED_LOG(color, ...) \
    PRINT_ANSI_COLOR(color); \
    printf(__VA_ARGS__); \
    PRINT_ANSI_COLOR(ANSI_COLOR_RESET); \
    printf("\n");


/**
 * @brief   以16进制打印数据
 * @param   [in] data   打印的数据数据
 * @param   [in] len    数据长度 
 * @note    输出格式 = %02X
*/
void print_hex_table(uint8_t *data, uint16_t len);

/**
 * @brief   彩色打印示例
 * @note    打印各个等级的信息以及16进制的格式化输出
*/
void debug_log_demo(void);

#endif

```
{% endfold %}

{% fold info@debug_log.c %}
```c
#include <stdio.h>
#include "debug_log.h"

const char *log_level_string[] = 
{
    [DBG_LOG_ERROR]     = "Error",
    [DBG_LOG_WARNING]   = "Warning",
    [DBG_LOG_INFO]      = "Info",
    [DBG_LOG_DEBUG]     = "Debug",
};

void print_hex_table(uint8_t *data, uint16_t len)
{
#if DBG_ENABLE
    // 打印标题行
    PRINT_ANSI_COLOR(ANSI_COLOR_CYAN);
    printf("     ");
    for (size_t i = 0; i < 0x10; i++)
    {
        printf("%02X ", (unsigned int)i);
    }
    PRINT_ANSI_COLOR(ANSI_COLOR_RESET);
    printf("\n");
	// 计算行数
	size_t rows = (len + 15) / 16; 
	for (size_t i = 0; i < rows; i++)
	{
		// 输出标题列
        PRINT_ANSI_COLOR(ANSI_COLOR_CYAN);
        printf("%04X ", (unsigned int)i * 16);
        // 输出数据列
        PRINT_ANSI_COLOR(ANSI_COLOR_MAGENTA);
		size_t j;
		for (j = 0; j < 16 && i * 16 + j < len; j++)
		{
			printf("%02X ", data[i * 16 + j]);
		}
		// 补齐空白列
		for (; j < 16; j++)
		{
			printf("   ");
		}
		printf("\n");
        PRINT_ANSI_COLOR(ANSI_COLOR_RESET);
	}
#endif
}

/**
 * @brief   彩色打印Demo
 */
void debug_log_demo(void)
{
#if DBG_ENABLE == 0
    printf("## Debug Print Disable, NO log print\n");
#else
    printf("Current log level is [%s]\n", log_level_string[DBG_LOG_LEVEL]);
#endif
    // 定义项目名称
    #define PROJECT_NAME_ART        " ____ ____ ____ ____ ____ ____ ____ ____ \n||L |||O |||G |||- |||D |||E |||M |||O ||\n||__|||__|||__|||__|||__|||__|||__|||__||\n|/__\\|/__\\|/__\\|/__\\|/__\\|/__\\|/__\\|/__\\|\n"
    // ** 自定义打印 不受级别控制
    ADVANCED_LOG(ANSI_COLOR_YELLOW, PROJECT_NAME_ART);
    // 中文打印测试
    DBG_LOGI("中文测试输出, 需要确保终端支持UTF-8的显示, 否则显示乱码\n");
    // 输出所有等级测试，可以尝试修改打印等级的宏以观察输出效果 DBG_LOG_LEVEL
    DBG_LOGD("The Log Level is %s", log_level_string[DBG_LOG_DEBUG]);
    DBG_LOGI("The Log Level is %s", log_level_string[DBG_LOG_INFO]);
    DBG_LOGW("The Log Level is %s", log_level_string[DBG_LOG_WARNING]);
    DBG_LOGE("The Log Level is %s", log_level_string[DBG_LOG_ERROR]);
    // 打印16进制
    uint8_t array[100] = {0};
    for (size_t i = 0; i < sizeof(array); i++)
    {
        array[i] = i;
    }
    print_hex_table(array, sizeof(array));
}

```
{% endfold %}

### 2.2 使用示例
![调用示例打印函数](call_demo.png)

![打印效果](demo_print.png)

## 3.进阶使用
### 3.1 关闭所有打印
通过`debug_log.h`中的宏`DGB_ENABLE`启用或禁用全局打印
![禁用全局打印的效果](log_disable.png)

### 3.2 按照等级打印
通过`debug_log.h`中的宏`DBG_LOG_LEVEL`控制打印等级。打印的等级从高到低的排序: **错误** > **警告** > **普通** > **调试**。假设指定打印等级为`DGB_LOG_WARNING`，即当前工程只会输出**警告**以及**错误**的信息，在工程产生大量的日志的时候十分有用。
![指定打印等级为警告及以上](log_warning.png)

### 3.3 行首说明
- 如果输出的等级是**调试**或**普通**，则行首的方括号表示的是**函数名称**
- ~~如果输出的等级是**警告**或**错误**，则行首的方括号表示的是**文件名(已经去除了文件路径)+代码行数+函数名称**~~，*Update*: 为了效果统一，所有打印等级的格式均一致，只有颜色区分，且行首的函数名称已经足够定位调试

{% note info %}
行首是可以自定义的，比如在FreeRTOS中可以加入系统当前的滴答数以粗略估计该打印的时间。其余同理。
{% endnote%}

![自定义行首](line_start.png)

## 参考
[^1]: [C彩色打印库的github地址](https://github.com/Orionxer/c_project/tree/main/debug_log)
