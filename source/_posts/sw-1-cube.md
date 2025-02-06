---
title: 1.SW-绘制正方体
date: 2024-1-1 00:00:05
categories:
- Solidworks
tags: [Solidworks, 3D, 结构]
type: solidworks
banner_img: /images/sw-1-cube.jpg
index_img: /images/sw-1-cube.jpg
excerpt: SolidWorks绘制一个最简单的正方体
---

## 1.软件界面
![hello-solidworks](sw_hello.png)
### 1.1 新建零件
主页选择**零件**选项卡并保存为`1.Hello_Part.SLDPRT`文件名
![新建保存零件](new_part.png)
### 1.2 新建草图
- 选择**草图**选项，点击**草图绘制**
- 鼠标左键点击**前视基准面**

![进入草图编辑](new_draft.gif)

### 1.3 绘制矩形
- 选择**中心矩形**(*选择**边角矩形**也可以，确保矩形的中心在原点即可*)
- 在原点处选择**中心点**
- 拖动绘制矩形,单击完成绘制矩形
- 按**Esc**键退出绘制矩形

![绘制矩形](new_rect.gif)
{% note success %}
**Esc**键退出绘制模式; **Ctrl + Z**撤销操作; **Ctrl + Y**还原操作;
{% endnote %}

### 1.4 添加约束
- 选择**智能尺寸**
- 单击任意一条边线编辑长度
- 输入长度(或者加入单位)
- 按**Esc**键退出智能尺寸

![添加约束](add_straint.gif)
{% note success %}
草图的所有线条颜色均为**黑色**说明草图**完全约束**，如果存在蓝色线条，说明草图未完全约束。
{% endnote %}

### 1.5 拉伸凸台基体
- 选择**拉伸凸台基体**
- **给定深度**处填入100
- 点击✅确定编辑

![拉伸凸台基体](add_depth.gif)

### 1.6 观测操作
- 按住鼠标**中键**，同时移动鼠标，即可观察该正方体
- **Ctrl + 7**恢复**等轴测**视角
- 点击**视图定向**, 选择需要正视的平面(**空格键**快速进入视图定向功能)
- **Ctrl + 鼠标中键**, 平面移动

![切换视图定向](switch_angle.gif)
