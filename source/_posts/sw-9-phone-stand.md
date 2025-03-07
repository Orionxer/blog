---
title: 9.SW-3D打印手机支架
date: 2024-07-08 06:30:37
categories:
- Solidworks
tags: [Solidworks, 3D, 结构]
type: solidworks
banner_img: /images/sw-9-phone-stand.jpg
index_img: /images/sw-9-phone-stand.jpg
excerpt: 使用SolidWorks设计一个手机支架，并使用3D打印机打印出来
---

{% note success %}
本篇文章使用SolidWorks设计一个手机支架，包含草图文本、草图图片，样条曲线，材料与质量属性，测量。并最终使用3D打印机打印模型。
{% endnote %}

## 1.基础特征
### 1.1 横截面
新建零件，选择**前视基准面**，绘制一个手机支架横截面

![横截面草图](cross_section.png)

### 1.2 拉伸凸台基体
拉伸凸台基体，宽度为**58mm**，两侧对称
![拉伸凸台基体](features.gif)

### 1.3 切除两边
- 选择底座前面进行草图绘制
- 左侧绘制一个倒三角形，原点上绘制一条直线，并设置为构造线(快捷键`Alt + C`)
- 选择镜像实体，选择镜像轴，提交编辑
- 标注尺寸为**52mm**
- 拉伸切除，点击**完全贯穿 - 两者**

![拉伸切除](features_cut.gif)

### 1.3 充电口
- 选择正斜面进行草图绘制
- 绘制半个梯形，原点上绘制构造线，镜像
- 分别设置尺寸，上底=23mm，下底=28mm，高=25mm，下底距离原点13mm
- 拉伸切除，方向1为**成形到一面**，选择背面；方向2为**完全贯穿**

![切除充电口](charge_cut.gif)

### 1.4 圆角/倒角
- 底座前侧增加**圆角**，半径**10mm**
- 转角处增加**倒角**，**8mm**，该位置应力最大，比较容易损坏
- 外圆转角分别增加圆角，**6mm**
- 托架处增加圆角，半径**3mm**；底部圆角**6mm**
- 两侧托架圆角**5mm**

![圆角倒角](round_corner.gif)

### 1.5 底座支点
- 底面进入草图绘制；靠右上方绘制一个圆，直径标注为15mm
- `Ctrl`键按下同时点击圆心与圆弧，选择**同心**
- 原点绘制一条横线，设置为构造线，在线上绘制一个圆，使第一个圆相等，标注圆心到原点距离56mm
- 选择镜像实体，选择第一个圆，镜像轴为构造线；拉伸凸台基体**1mm**

![底座支点](bottom.gif)

### 1.6 背部挖孔
- 背面进入草图绘制；原点绘制一条竖线，设置为构造线
- 绘制直槽口，长度20mm，宽度20mm，底部圆心到原点距离32mm
- 拉伸切除，选择**成形到一面**，选择里面

![背部挖孔](back.gif)

## 2.草图文本
- 背面进入草图绘制，点击文本
- 属性栏输入文本，比如`French Bulldog`，移动位置到靠底部的位置
- 改变字体样式以及大小，根据实际情况调整(比如选择Adobe Gothic，大小5mm)
- 拉伸切除0.3mm
- 补充漏掉的两个圆角5mm

![草图文本](draft_text.gif)

## 3.草图图片
- 绘制两个同心圆，外圆直径30mm，距离内部圆1mm
- 选择正面进入草图编辑，选择**工具** -> 草图工具 -> 草图图片
- 调整透明度以及大小，移动到合适的位置，确定

![草图文本](draft_picture.gif)

## 4.样条曲线
- 点击样条曲线，沿着曲线一个个点绘制，尽量贴合
- 如果遇到不够贴合的情况，右键选择**显示控制多边形**，拖动多边形的端点使曲线更加贴合
- 拉伸切除，注意草图被线条分割成了不同的轮廓，在**所选轮廓**中选择所有的轮廓，确定

![草图文本](curse_line.gif)

![选择轮廓拉伸切除](select_cut.gif)

## 5.补充圆角
- 补充4个圆角3mm
- 所有边线设置倒角1mm

![补充圆角倒角](complementary.gif)

## 6.导入3D打印机
{% note info %}
这里使用拓竹A1 Mini打印机，精度0.4mm，使用PLA材质打印，其官方的切图软件为**Bambu Studio**
{% endnote %}
- 另存为工程为.STL文件
- 打开Bambu Studio，导入该STL文件
- 点击支撑。勾选**开启支撑**，类型选择**树状(自动)**
- 点击切片单盘，打印单盘，发送打印任务

![3D打印](3d_print.gif)

## 7.打印效果
![实际打印效果](result.png)

