---
title: 5.SW-特征成型与设计树
date: 2024-01-20 22:41:54
categories:
- Solidworks
tags: [Solidworks, 3D, 结构]
type: solidworks
banner_img: /images/sw-5-features.png
index_img: /images/sw-5-features.png
excerpt: SolidWorks特征成型与设计树
---

{% note success %}
SolidWroks特征成型是将二维草图变为三维实体的过程。绘制一个正方体[^1]就使用到了了**拉伸凸台基体**的功能，是一个最简单的特征概念。
{% endnote %}

## 1.特征成型介绍
![特征成型基础功能](basic.png)

## 2.拉伸凸台基体与设计树
### 2.1 绘制一个半熟芝士
- 草图选择上视基准面
- 绘制水平槽口
- 拉伸凸台基体: 方向1给定深度20mm，方向2给定深度10mm

![半熟芝士](cheese.gif)

### 2.2 绘制类似TypeC插口
- 选择实体顶面绘制草图
- 选择转换实体引用
- 方向1给定深度55mm
- 勾选薄壁特征，单击反向，选择单向，输入5mm

![类似TypeC插口](typec.gif)

{% note info %}
修改草图尺寸，其对应的凸台基体也会对应修改变化
{% endnote %}

## 参考
[^1]: [SolidWorks绘制正方体](https://blog.gogo.uno/2023/12/31/sw-1-cube/)

