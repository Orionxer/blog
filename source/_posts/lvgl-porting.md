---
title: LVGL9-移植
date: 2025-10-03 11:14:01
categories:
- LVGL
type: LVGL
banner_img: /images/lvgl.jpg
index_img: /images/lvgl.jpg
excerpt: LVGL9.4运行于Windows环境下，并移植Gui Guider导出的Demo
---


## 1.环境
检查`gcc`,`g++`, `make`, `cmake`编译环境，如果不具备则安装mingw64和cmake

### 1.1 Mingw64下载安装
下载已经编译好的mingw64版本[mingw-build-binaries](https://github.com/niXman/mingw-builds-binaries/releases),选择类似于`x86_64-15.2.0-release-posix-seh-ucrt-rt_v13-rev0.7z`的版本下载并解压到D盘下，例如`D:\mingw64`,则PATH的系统环境变量应该添加`D:\mingw64\bin`，注意，在该目录下把`mingw32-make.exe`复制一份并重命名为`make.exe`。

### 1.2 CMake下载安装

下载cmake的安装包[cmake](https://cmake.org/download/)，选择类似于`cmake-4.1.2-windows-x86_64.zip`的版本，解压到D盘下，例如`D:\cmake-4.1.2-windows-x86_64`,则PATH的系统环境变量应该添加`D:\cmake-4.1.2-windows-x86_64\bin`。

### 1.3 Vcpkg下载安装
在D盘下克隆vcpkg的仓库，[教程：通过 CMake 安装和使用包](https://learn.microsoft.com/zh-cn/vcpkg/get_started/get-started?pivots=shell-powershell)
```sh
git clone https://github.com/microsoft/vcpkg.git
```
进入vcpkg目录，执行启动脚本
```sh
cd vcpkg; .\bootstrap-vcpkg.bat
```
环境变量PATH添加`D:\vcpkg`

### 1.4 检查环境
```sh
gcc --version
```
```sh
g++ --version
```
```sh
make --version
```
```sh
cmake --version
```
```sh
vcpkg --version
```

## 2.安装SDL2环境
```sh
vcpkg install sdl2:x64-mingw-static
```
则安装好的SDL2的路径应该是`D:\vcpkg\installed\x64-mingw-static\share\sdl2`

## 3.下载LVGL
```sh
git clone --recursive https://github.com/lvgl/lv_port_pc_vscode
```
### 3.1 修改CMakeLists.txt
修改根目录下的`CMakeLists.txt`，在`# Find and include SDL2 library`下面一行引入SDL2库，例如
```cmake
# Find and include SDL2 library
# 引入SDL2库
set(SDL2_DIR "D:/vcpkg/installed/x64-mingw-static/share/sdl2")
find_package(SDL2 REQUIRED)
```

### 3.2 使用CMake编译
创建并进入build目录
```sh
mkdir build && cd build
```
生成Makefile
```sh
cmake -G "MinGW Makefiles" ..
```
编译项目
```sh
make -j16
```

### 3.3 运行
进入`bin`目录找到`main.exe`双击运行
```sh
cd ../bin; ./main.exe
```

## 4.优化

### 4.1 修改运行尺寸
`main.c`中找到`sdl_hal_init`函数，修改尺寸
```c
sdl_hal_init(720, 1280);
```

### 4.2 鼠标残影
`hal.c`中找到鼠标初始化相关代码，注释掉，否则会有鼠标残影
```c
#if 0 // 注释鼠标图片，否则会有重影
  /*Declare the image file.*/
  LV_IMAGE_DECLARE(mouse_cursor_icon); 
  lv_obj_t * cursor_obj;
  /*Create an image object for the cursor */
  cursor_obj = lv_image_create(lv_screen_active()); 
  /*Set the image source*/
  lv_image_set_src(cursor_obj, &mouse_cursor_icon);           
  /*Connect the image  object to the driver*/
  lv_indev_set_cursor(mouse, cursor_obj);             
#endif
```

### 4.3 打开监控
`lv_conf.h`中确保以下宏定义启用
```c
#define LV_USE_SYSMON   1
#define LV_USE_PERF_MONITOR 1
#define LV_USE_MEM_MONITOR 1
```

### 4.4 提升帧率
`lv_conf.h`中修改以下宏定义，16ms刷新一次，即60FPS
```c
#define LV_DEF_REFR_PERIOD  16
```

重新编译运行，记得关闭程序再编译，否则提示`Permission denied`错误

## 5.Gui Guider导出

> 以Gui Guider中的Demo: SmartAppliance为例进行导出

### 5.1 生成Project
- 点击`New`，在LVGL9.2.1的标签下选择`Simulator`，选择`SmartAppliance`，点击`Create Project`
- `Project Directory`修改为项目保存路径
- 其他参数自定义，分辨率可以保持默认的`480*272`
- 点击`Create`确定
- 尝试点击右上角的编译按钮，选择`C`进行编译运行

### 5.2 导出代码
- 点击`Project`
- 移动到`Export Code`
- 选择`Zephyr`
- 弹窗选择保存路径
导出后的目录结构如下
```sh
.
├── CMakeLists.txt
├── patches
├── prj.conf
└── src
    ├── custom
    │   ├── custom.c
    │   ├── custom.h
    │   ├── lv_conf_ext.h
    │   ├── ui_Aircon.c
    │   ├── ui_Aircon.h
    │   ├── ui_Oven.c
    │   └── ui_Oven.h
    ├── generated
    │   ├── events_init.c
    │   ├── events_init.h
    │   ├── gui_guider.c
    │   ├── gui_guider.h
    │   ├── guider_customer_fonts
    │   ├── guider_fonts
    │   ├── images
    │   ├── setup_scr_scrAircon.c
    │   ├── setup_scr_scrHome.c
    │   ├── setup_scr_scrHood.c
    │   ├── setup_scr_scrOven.c
    │   ├── widgets_init.c
    │   └── widgets_init.h
    └── main.c
```
其中`custom`和`generated`文件夹下的源文件需要加入编译。

### 5.3 复制文件
在成功运行lvgl9的官方Demo后，把`custom`和`generated`目录复制到`src`目录下，结构如下
```sh
src/
├── custom
├── freertos
├── freertos_main.c
├── generated
├── hal
├── main.c
└── mouse_cursor_icon.c
```

### 5.4 修改文件
#### 5.4.1 main.c
在引入头文件部分添加
```c
#include "generated/gui_guider.h"
#include "generated/events_init.h"
```
再添加guider_ui结构体
```c
lv_ui guider_ui;
```
在main函数里的初始化官方demo处，替换为ui和events初始化，例如
```c
setup_ui(&guider_ui);
events_init(&guider_ui);
// lv_demo_widgets();
```

#### 5.4.2 其他文件
把所有的源文件中的`lv_animimg_set_src`的最后一个布尔值参数删除，删除后类似如下
```c
lv_animimg_set_src(ui->scrAircon_aimgAirconSwing, (const void **) scrAircon_aimgAirconSwing_imgs, 15);
```
- 所有函数`lv_animimg_set_playback_time`替换为`lv_animimg_set_reverse_duration`
- 所有函数`lv_animimg_set_playback_delay`替换为`lv_animimg_set_reverse_delay`

#### 5.4.3 添加编译规则
根目录下的`CMakeLists.txt`文件增加编译规则，大概37行后面添加新的编译路径
```cmake
# Main include files of the project
include_directories(${PROJECT_SOURCE_DIR}/main/inc)
# 新增的编译路径
include_directories(${PROJECT_SOURCE_DIR}/src)
include_directories(${PROJECT_SOURCE_DIR}/src/custom)
include_directories(${PROJECT_SOURCE_DIR}/src/generated)
```
`set(MAIN_SOURCES src/mouse_cursor_icon.c src/hal/hal.c)`修改为
```cmake
file(GLOB_RECURSE GENERATED_SOURCES 
    src/generated/*.c
    src/generated/guider_customer_fonts/*.c
    src/generated/guider_fonts/*.c
    src/generated/images/*.c
)
file(GLOB CUSTOM_SOURCES src/custom/*.c)
set(MAIN_SOURCES src/mouse_cursor_icon.c src/hal/hal.c ${GENERATED_SOURCES} ${CUSTOM_SOURCES})
```
重新编译运行

## 6.直接克隆
克隆已经移植好的仓库,符合环境要求则可直接编译运行
```sh
git clone --recursive https://github.com/orionxer/lv_port_pc_vscode
```
