---
title: 2.SW-绘制草图
date: 2024-01-05 22:58:08
categories:
- Solidworks
tags: [Solidworks, 3D, 结构]
banner_img: /images/sw-2-draw-draft.jpg
index_img: /images/sw-2-draw-draft.jpg
excerpt: SolidWorks绘制草图工具,包括直线/矩形/圆/槽/圆弧/圆角/点
---

{% note success %}
SolidWorks的草图绘制逻辑是**先绘制图形，再添加尺寸约束**，使草图达到完全约束的状态。
{% endnote %}

## 1.直线
### 1.1 绘制直线
- 选择**直线**工具
- 点击**原点**
- 拖动绘制直线
- **Esc**退出绘制直线

![绘制直线](new_line.gif)

### 1.2 添加约束
![添加约束](line_straint.gif)
如果**几何关系**存在冲突的规则，则删除冲突的规则即可。
{% note info %}
确保草图完成编辑时处于**完全定义**的状态，此时所有草图应该确保为**黑色**(包括点)，如果*未完全约束*则显示为*蓝色*
{% endnote %}
![完全定义](all_straint.png)

## 2.矩形
### 2.1 绘制矩形
- 选择并绘制**边角矩形**

![绘制矩形](draw_rect.gif)

### 2.2 添加约束
- 选择**智能尺寸**
- 点击矩形的边长设置尺寸

![矩形添加约束](rect_straint.gif)
{% note info %}
进入**尺寸标注**功能的另一种方法：鼠标按住**右键**向上移动
{% endnote %}
![尺寸标注](size_label.png)

## 3.圆
### 3.1 中心圆
![绘制中心圆](draw_circle_center.gif)
### 3.2 周边圆
**周边圆**即三点绘制一个圆
![绘制周边圆](draw_circle_side.gif)

## 4.槽
![绘制槽](draw_tank.gif)

## 5.圆弧
![绘制切线弧](draw_arc.gif)

## 6.圆角与倒角
![绘制圆角与倒角](draw_corners.gif)

## 7.点
![点](point.png)
