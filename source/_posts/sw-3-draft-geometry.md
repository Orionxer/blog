---
title: 3.SW-草图几何关系与编辑
date: 2024-01-11 06:08:51
categories:
- Solidworks
tags: [Solidworks, 3D, 结构]
banner_img: /images/sw-3-draft-geometry.jpg
index_img: /images/sw-3-draft-geometry.jpg
excerpt: SolidWorks绘制草图几何关系
---

{% note success %}
SolidWroks草图几何关系包括：**水平、竖直、重合、中点、相切、平行、相等、共线、对称**等
{% endnote %}

## 1.草图几何关系

![常见的草图几何关系](geometry.png)

### 1.1 垂直
- 绘制两条直线(其中第一条设置为水平线)，另一条任意绘制
- 选中两条直线
- 在左侧的**添加几何关系**中选择**垂直**关系

![设置两条直线的垂直关系](geometry_vertical.gif)

### 1.2 平行

![设置两条直线的平行关系](parallel.gif)
{% note info %}
如果是三条不共线的直线添加平行特征，则以第二条为基准来平行
{% endnote %}

### 1.3 相切

![设置圆与直线相切](tangency.gif)

### 1.4 对称
- 绘制一条直线，设置为**构造线**
- 绘制第一个圆，使圆与直线相切
- 绘制第二个圆，选中直线与两个圆，点击**对称**

![设置两个圆相对于构造线对称](symmetry.gif)

## 2.裁剪实体
{% note info %}
裁剪实体用于删除不必要的线段部分，最常用到的是**强劲裁剪**和**裁剪到最近端**
{% endnote %}
### 2.1 强劲裁剪
#### 2.1.1 裁剪(删除线段)
- 绘制两条相交的直线
- 选择**裁剪实体**功能
- 选择属性栏的**强劲裁剪**
- 鼠标**左键按住不放划过线段**部分，能够看到线段被裁剪至最近的端点，即删除了线段

![强劲裁剪-删除线段](tailor_delete.gif)

#### 2.1.2 延伸(延长线段)
- 点击线段不放，滑动鼠标即可延长线段

![强劲裁剪-延伸线段](tailor_extend.gif)
### 2.2 裁剪到最近端
- 与强劲裁剪类似，只是没有了延伸的功能，单击线段执行删除效果

![裁剪到最近端](tailor_delete_click.gif)

## 3.转换实体引用
{% note info %}
转换实体引用可以简单理解为**投影**，将需要使用的到线段(或者其他几何图形)投影到需要的平面上继续使用
{% endnote %}
### 3.1 转换圆柱体顶面
- 绘制一个圆(在这里可以理解为底面的圆)
- 点击**拉伸凸台基体**，任意设置一个深度，圆柱就生成了
- 重新进入草图绘制，选择圆柱顶面，此时可以对顶面可以进行草图编辑
- 点击**转换实体引用**，点击圆的**外部轮廓**，属性栏点击**确定**(使转换实体引用生效)

![转换实体引用示例](draw_column.gif)

### 3.2 观察圆柱体
- 点击**显示类型**，选择**线架图**
- 选择**带边线上色**则恢复默认视角

![查看圆柱体](view_column.gif)

## 4.等距实体
- 绘制一个圆
- 点击**等距实体**
- 属性栏填入等距距离，选择**反向**或**双向**

![绘制等距实体](isometry.gif)

## 5.添加长度/角度约束
- 在**智能尺寸**功能下，点击第一条线后继续点击第二条线，就能设置两条线的角度

![设置角度约束](line_angle.gif)