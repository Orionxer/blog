---
title: 生成ASCII艺术字
date: 2024-02-22 01:22:25
categories:
- Python
tags: [Python, C/C++, ANSI转义序列]
banner_img: /images/generate-ascii-art.jpg
index_img: /images/generate-ascii-art.jpg
excerpt: Python生成ASCII艺术字以及生成C代码
---

{% note success %}
使用Python生成`ASCII`艺术字，可以选择是否输出颜色(需要终端支持ANSI转义序列)。同时生成包含`ASCII`艺术字的C代码。该方法可以用于生成个性化的提示信息，比如**系统开机提示**。*暂时不支持生成中文。*
{% endnote %}

## 1.pyfiglet模块
需要满足Python或者Python3运行环境
### 1.1 安装
pip安装pyfiglet
```sh
pip install pyfiglet
```
### 1.2 测试命令
打印一个ASCII艺术字体，风格为`banner3`，内容为`RTOS`
```sh
pyfiglet -f banner3 RTOS
```
![pyfiglet命令测试结果](cmd_test.png)
### 1.3 风格
列出所有可用风格，但是该命令只能获取到风格名称，查看具体的效果需要去`figlet`网站[^1]
```sh
pyfiglet -l
```
列出所有包含`banner`的风格
```sh
pyfiglet -l | grep banner
```
## 2.Python代码
### 2.1 示例代码
以下提供了一个Demo可以直接运行，该Demo会根据选择的风格以及内容生成并打印ASCII艺术字体，并生成C代码，同级目录会自动生成一个`ascii.c`文件。且`ascii.c`可以直接运行，其打印的艺术字体与Python的打印一致。该Demo也可以去github下载[^2]。
{% fold info@Python示例代码 %}
```py
import pyfiglet

# 选择ASCII艺术字体风格
font = "banner3"
# 需要生成的内容
text = "RTOS"
# 是否需要支持ANSI转义序列
enable_color = 0
# 选择输出颜色
color = "yellow"
# ANSI颜色映射表(可以根据ANSI规范自行)
ansi_map = {
    "red"       : "\x1b[31m",
    "green"     : "\x1b[32m",
    "yellow"    : "\x1b[33m",
    "blue"      : "\x1b[34m",
    "magenta"   : "\x1b[35m",
    "cyan"      : "\x1b[36m",
    "reset"     : "\x1b[0m"
}
# 目标字符串
result = pyfiglet.figlet_format(text, font=font)
# 如果启用了颜色输出则加入ANSI颜色映射
if enable_color:
    result = ansi_map[color] + result + ansi_map["reset"]
# 测试打印效果
print(result)

# 禁止转义
result = repr(result)
# 删除单引号'
result = result.replace("'", "")

# 生成C代码内容
c_code = """
#include <stdio.h>
// 目标字符串
const char *str = "%s";
int main() {
    printf("%%s\\n", str);
    return 0;
}
"""

# 将目标字符串连接到final_c_code中
final_c_code = c_code % result
# 保存文件
with open("ascii.c", "w") as file:
    file.write(final_c_code)

print("C source file saved!")

```
{% endfold %}

### 2.2 运行
{% note primary %}
建议在VSCode里安装`Code Runner`插件，可以一键运行Python代码以及C代码。注意，`Code Runner`需要打开`Run in Terminal`选项，否则默认的`OUTPUT`控制台不支持ANSI的序列化显示。
{% endnote %}

![打开Run in Terminal](setting.png)
#### 2.2.1 运行Python文件
将Python示例程序保存为`generate.py`，在该文件处右键`Run Code`，观察终端输出的结果，以及同级目录的`ascii.c`文件是否生成
![运行Python程序](generate.gif)
#### 2.2.2 运行C文件
右键运行C文件，观察终端输出的结果
![运行C程序](run_c.gif)

## 3. 进阶

### 3.1 启用颜色
尝试启用颜色输出，设置`enable_color = 1`，颜色根据程序里定义好的`ansi_map`自由选择
![启用颜色](enable_color.gif)

### 3.2 修改风格
![修改风格](change_style.gif)
## 参考

[^1]: [figlet在线查看ASCII艺术字体](http://www.figlet.org/examples.html)
[^2]: [Python生成示例github仓库](https://github.com/Orionxer/c_project/tree/main/ascii_art)
