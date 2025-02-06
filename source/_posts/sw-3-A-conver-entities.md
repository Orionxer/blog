---
title: 3.A-SW-转换实体引用
date: 2024-01-14 01:29:48
categories:
- Solidworks
tags: [Solidworks, 3D, 结构]
type: solidworks
banner_img: /images/sw-3-A-conver-entities.png
index_img: /images/sw-3-A-conver-entities.png
excerpt: SolidWorks演示【转换实体引用】功能
---

{% note success%}
**转换实体引用**简单理解为**投影**，将需要使用的到线段(或者其他几何图形)投影到需要的平面上继续使用。以画出俄罗斯方块的**L**和**凸**形状为例去演示转换实体引用的用法。
{% endnote %}

## 1.绘制缺口方块
### 1.1 绘制带缺口的正方形
![绘制带缺口的正方形](rect_gap.gif)

### 1.2 设置尺寸
正方形边长为100mm,缺口为50mm
![设置尺寸](rect_gap_straint.gif)

### 1.3 拉伸凸台基体
设置深度为50mm
![设置深度](rect_gap_depth.gif)

## 2.绘制成L形状
- 点击顶面(靠左侧)
- 点击**转换实体引用**
- 拉伸凸台基体为50mm

![绘制L形状](convert_entitis_L.gif)

## 3.删除的凸台拉伸和草图

![删除的凸台拉伸和草图](convert_entitis_delete.gif)

## 4.绘制成凸形状
- 点击左侧面，进入草图编辑
- 鼠标**中键**旋转视图到右侧，单击右侧面
- 点击**转换实体引用**，此时观察到右侧面的正方形投影到左侧面的草图
- 拉伸凸台基体为50mm

![绘制凸形状](convert_entitis_bulge.gif)


