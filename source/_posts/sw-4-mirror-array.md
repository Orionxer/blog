---
title: 4.SW-草图镜像与阵列
date: 2024-01-17 07:31:14
categories:
- Solidworks
tags: [Solidworks, 3D, 结构]
type: solidworks
banner_img: /images/sw-4-mirror-array.jpg
index_img: /images/sw-4-mirror-array.jpg
excerpt: SolidWorks草图关于镜像与阵列的功能
---

## 1.镜像
{% note info %}
注意：镜像轴必须是一条直线
{% endnote%}

- 草图绘制一个圆，在圆的左侧绘制一条直线
- 选择**镜像实体**
- 左侧属性栏选择**要镜像的实体**，再点击圆
- 左侧属性栏选择**镜像轴**，再点击直线，此时能够看到直线左侧生成了圆的镜像预览
- 点击确认生成镜像圆

![草图镜像](mirror.gif)

## 2.阵列
### 2.1 线性草图阵列
- 选择**线性草图阵列**
- 选择右侧圆，即需要阵列的实体
- 属性栏在X轴填入需要阵列的**数量**以及阵列的**距离**
- 属性栏Y轴同理，不过需要在数量上设置大于1的数字才能编辑阵列距离

![线性草图阵列](line_array.gif)

### 2.2 圆周草图阵列
- 绘制一个点
- 选择**圆周草图阵列**
- 左侧属性栏的**参数**选择刚刚绘制的点
- **要阵列的实体**选择左侧圆
- 输入需要阵列的数量以及角度

![圆周草图阵列](circle_array.gif)
