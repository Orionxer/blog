---
title: Linux搭建C开发环境
date: 2024-02-05 01:03:30
categories:
- C/C++
tags: [Linux, VSCode, C/C++]
banner_img: /images/vscode-develop-c.jpg
index_img: /images/vscode-develop-c.jpg
excerpt: 基于Linux搭建 VSCode + CMake + C开发环境
---

{% note success %}
本篇文章基于WSL2 Ubuntu，使用CMake搭建 VSCode + C 的编译调试开发环境，**本质上适用于所有Linux系统**。
也可以从github上克隆示例项目[^1]到本地，直接启动调试。
{% endnote %}

## 1.安装编译调试环境
```sh
sudo apt install cmake build-essential gcc gdb -y
```
## 2.初始化工程结构
### 2.1 终端创建目录以及文件 
```sh
# 创建工程目录并进入
mkdir basic && cd basic
# 工程根目录创建.vscode文件夹
mkdir .vscode
# 创建VSCode编译调试的配置文件
touch .vscode/c_cpp_properties.json
touch .vscode/launch.json
touch .vscode/tasks.json
# 创建编译相关文件夹
mkdir include
mkdir source
mkdir build
# 创建主函数文件
touch main.c
# 创建CMake配置文件
touch CMakeLists.txt
```

{% note info %}
c_cpp_properties.json也可以通过VSCode自动生成: 在VSCode中`Ctrl + Shift + P`调出控制命令台，输入并选择`C/C++ Edit Configurations(JSON)`，VSCode会自动生成合法的配置。
{% endnote %}
### 2.2 打开VSCode工作区
```sh
code .
```
### 2.3 复制内容到对应文件

{% fold info@c_cpp_properties.json%}
```json
{
    "configurations": [
        {
            "name": "Linux",
            "includePath": [
                "${workspaceFolder}/**"
            ],
            "defines": [],
            "compilerPath": "/usr/bin/gcc",
            "cStandard": "c17",
            "cppStandard": "gnu++14",
            "intelliSenseMode": "linux-gcc-x64",
            "configurationProvider": "ms-vscode.cmake-tools"
        }
    ],
    "version": 4
}
```
{% endfold %}


{% fold info@launch.json%}
```json
{
    // 更多调试设置参考：https://code.visualstudio.com/docs/cpp/launch-json-reference
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Project debug (C/C++ GDB)",
            "type": "cppdbg",
            "request": "launch",
            "program": "${workspaceFolder}/debug/main",
            "args": [],
            "cwd": "${workspaceFolder}",
            "stopAtEntry": true, // 启动调试时在main函数暂停
            "externalConsole": false,
            "preLaunchTask": "Build For Debug",
            "linux": {
                "MIMode": "gdb",
                "miDebuggerPath": "/usr/bin/gdb"
            }
        }
    ]
}
```
{% endfold %}

{% fold info@tasks.json%}
```json
{
    // 更多任务设置参考：https://go.microsoft.com/fwlink/?LinkId=733558
    "version": "2.0.0",
    "tasks": [
        {
            "label": "Generate Debug",
            "type": "cppbuild",
            "command": "cmake",
            "args": [
                "-B",
                "debug",
                "-D",
                "CMAKE_BUILD_TYPE=Debug",
                "-G",
                "Unix Makefiles",
                "${workspaceFolder}"
            ],
            "options": {
                "cwd": "${workspaceFolder}"
            },
            "problemMatcher": [
                "$gcc"
            ],
            "group": {
                "kind": "build",
                "isDefault": false
            },
        },
        {
            "label": "Build For Debug",
            "type": "cppbuild",
            "command": "cmake",
            "args": [
                "--build",
                "debug"
            ],
            "options": {
                "cwd": "${workspaceFolder}"
            },
            "problemMatcher": [
                "$gcc"
            ],
            "dependsOn": [
                "Generate Debug"
            ],
            "group": {
                "kind": "build",
                "isDefault": true
            },
        }
    ]
}
```
{% endfold %}

{% fold info@main.c %}
```c
#include "stdio.h"

int main(void)
{
    printf("## Hello World\n");
    return 0;
}
```
{% endfold %}

{% fold info@CMakeLists.txt %}
```txt
# CMake最低版本号要求
cmake_minimum_required(VERSION 3.0)

# 配置项目名
project(hello_c)

# 设置包含目录
include_directories(include)

# 递归遍历获取项目的所有源文件列表
file(GLOB_RECURSE SRC_LIST FOLLOW_SYMLINKS main.c source/*.c)

# 生成可执行文件 main，后面是源码列表
add_executable(main ${SRC_LIST})
```
{% endfold %}

### 2.4 工程目录结构参考
以上准备工作结束后，工程的目录结构如下图所示，其中
- `include`文件夹放置**头文件**，例如：`sha256.h`
- `source`文件夹放置**源文件**，例如：`sha256.c`

![工程目录结构](directory.png)

## 3.编译调试
`F7`编译，`F5`启动调试(`F5`包含编译的过程)，按照`launch.json`的`StopAtEntry:true`的定义，程序应该暂停在`main`
![启动调试](f5_debug.png)


## 4.调试入门
{% note primary %}
所有语言以及IDE在调试的逻辑上基本是一致的，甚至连调试的快捷键也可能相同。比如MDK和VSCode的调试快捷键完全相同，虽然两者完全不具备可比性。VSCode官方介绍的Debugging用法[^2]。
{% endnote %}
### 4.1 断点
断点作为调试中最重要的功能，常用的有三种，普通断点、条件断点和日志断点
#### 4.1.1 普通断点
在一行语句上创建断点，启动调试后，程序会在该语句前暂停，即**断点所在的行未被执行**。以下图为例，程序执行到该断点处，b的值为0，说明语句`b = debug_func(a);`未被执行。继续往下执行，则b的值变为45，说明语句执行了。
![普通断点 - Breakpoint](normal_breakpoint.gif)

#### 4.1.2 条件断点
`Conditional Breakpoint`，条件断点，当程序符合预设的条件时，暂停在断点处，其标志是断点符号中有个**等号**。条件断点一般有两种情况，符合**表达式**或者**命中次数**。命中次数可以简单理解为该语句执行了多少遍。以下图为例，创建一个条件断点，`Expression`表达式`i > 5`与`Hit Count`命中次数`5`是等价的。
![条件断点 - Expression - 表达式](conditional_breakpoint_expression.gif)
![条件断点 - Hit Count - 命中次数](conditional_breakpoint_hit_count.gif)
{% note warning %}
GDB调试器不一定支持`Hit Count`断点，启动调试后提示`Breakpoints of this type are not supported by the debugger`，该断点退化成普通断点。
增加一个变量，统计函数的运行次数，使用`Expression`的条件断点能够达到与`Hit Count`一样的效果。
{% endnote %}

#### 4.1.3 记录点
当程序不方便暂停调试时，可以添加`记录点`，或者叫`Logpoint`。其符号是**方块形状**，在`DEBUG CONSOLE`控制台输出日志。*如果称其为日志断点则不是很合适，因为程序没有暂停运行，不符合断点的概念*
![记录点 - Logpoint](logpoint.gif)

#### 4.1.4 内联断点
内联断点，即`Inline Breakpoint`，用于调试一行代码中包含多个语句的情况。以下图为例，在`for`循环语句中，虽然只有一行代码，但是该行包含了3个语句：`int i = 0`和`i < a`以及`i++`。因此内联断点需要在**指定的列**前添加并显示。
{% note warning %}
在实际的工程项目中，如果工程遵守合理的编码规范的情况下，内联断点是基本用不上的。
{% endnote %}
![内联断点 - Inline Breakpoint](inline_breakpoint.gif)

#### 4.1.5 函数断点
创建函数断点：选择`Run`->`New Breakpoint`->`Function Breakpoint`，输入函数的名称，断点创建成功。该方式*可以但没必要*，不如直接在函数的入口打一个普通断点来的方便。

### 4.2 监控变量
#### 4.2.1 基础变量
`Watch`窗口可以查看所有类型的变量
- 数值类变量`int`,`float`,`double`
- 字符类型，先显示十进制的数值再显示`ASCII`对应的字符
- 字符串(本质上是字符串指针)，先显示地址再显示字符串的值
- 自定义的结构体以及枚举，枚举会显示枚举名称

![基础变量显示](basic_variable.png)

#### 4.2.2 不同进制
以int变量`a`为例，`a`默认以十进制的值显示，`a,h`以十六进制显示, `a,b`二进制，`a,o`八进制
![变量以不同进制显示](hexadecimal.png)

#### 4.2.3 表达式
监控变量窗口可以直接计算出表达式的结果
![计算表达式](watch_expression.png)

#### 4.2.4 指针
如果监控一个`void`类型的指针，监控窗口只会显示指针地址，使用**强制类型转换**才能正确显示字符串或结构体或数组的具体值。下图展示`void`类型的指针转换为字符串指针或者结构体指针用法
![强转void指针为具体类型](cast_pointer.gif)

强转数组则有一点区别，因为只强转数组的类型是不够的，还需要指定长度。假设需要强制指针`array_ptr`成长度为10的int数组，则表达式如下
```c
*(int(*)[10])array_ptr
```
![强转数组指针](cast_array.gif)

### 4.3 调用堆栈
以一个递归为例子，从1累加到5，预期进入5次递归函数。左侧的`CALL STACK`就会显示当前程序的调用栈。切换栈的深度时，观察`arg`的值

![调用堆栈](call_stack.gif)

### 4.4 快捷键
{% note primary %}
熟练常用的调试快捷键非常重要，在实际项目中的效率远高于按钮点击的方式。
{% endnote %}
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

| 快捷键 |   英文    | 中文 |        描述        |
| :----: | :------: | :--: | :-------------:    |
|  `F5`  | Continue | 继续  | 继续运行到下一个断点 |
|  `F10` | Step Over | 单步跳过  | 执行一行代码 |
|  `F11` | Step In | 单步执行  | 进入函数 |
|  `Shift + F11` | Step Out | 单步停止  | 跳出函数 |
|  `F9` | Toggle Breakpoint | 切换断点  | 添加或者删除断点 |
</div>

#### F5
![F5 - Continue - 继续](continue.gif)
#### F10
![F10 - Step Over - 单步跳过](step_over.gif)
#### F11
![F11 - Step In - 单步执行](step_in.gif)
#### Shift + F11
![Shift F11 - Step Out- 单步停止](step_out.gif)

## 5.导入C文件
根据当前的CMake规则，所有在`include`和`source`文件夹下的C文件会自动加入编译链接。以导入`sha256`库[^4]为例子，计算一个从1到10的数组的哈希值
{% fold info@计算哈希示例程序%}
```c
#include "stdio.h"
#include "sha256.h"

int main(void)
{
    uint8_t raw[10] = {0};
    for (size_t i = 0; i < sizeof(raw); i++)
    {
        raw[i] = i + 1;
    }
    uint8_t result[32] = {0};
    sha256_context context;
    // 初始化哈希
    sha256_init(&context);
    // 计算哈希
    sha256_hash(&context, raw, sizeof(raw));
    // 结束哈希
    sha256_done(&context, result);
    printf("The Hash result is: ");
    // 打印哈希结果
    for (size_t i = 0; i < sizeof(result); i++)
    {
        printf("%02x", result[i]);
    }
    printf("\n");
    return 0;
}
```
{% endfold %}

![计算哈希](hash.gif)

使用在线哈希计算工具验证计算结果[^5]。

![在线哈希工具验证结果](hash_result.png)

## 6.终端编译运行
```sh
# 工程目录进入build文件夹
## 如果没有build文件夹则创建一个
cd build

# CMake以及make编译
cmake ../ && make

# 运行可执行文件
./main

# 重新编译
## 1.清空build目录
rm -r ./*
## 2.再次编译
cmake ../ && make
```
### 6.1 测试结果
![终端编译运行结果](terminal.png)

## 参考
[^1]: [Linux编译调试C的github地址](https://github.com/Orionxer/c_project)
[^2]: [VSCode官方的Debugging用法](https://code.visualstudio.com/docs/editor/debugging)
[^3]: [VSCode 调试 WSL C/C++ 项目](https://learnku.com/articles/67523)
[^4]: [SHA256 for C github地址](https://github.com/ilvn/SHA256)
[^5]: [在线哈希值计算](https://www.lddgo.net/encrypt/hash)
