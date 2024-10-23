---
title: Linux命令-1
date: 2024-08-04 01:56:32
categories:
- Linux
tags: [Linux]
banner_img: /images/linux-cmd-1.jpg
index_img: /images/linux-cmd-1.jpg
excerpt: Linux基础命令介绍以及举例
---

{% cb 合理安排文章篇幅后修改excerpt, false, false, false %}

## 压缩图片
{% note success %}
使用**ImageMagick**命令压缩PNG，JPEG图片
{% endnote %}

以本站博客为例，每篇博客均有配图，如果图片体积过大，则页面会加载缓慢。在Linux下使用**ImageMagick**命令压缩图片，在不修改图片的分辨率前提下，尽可能降低图片体积且确保图像质量不会损失过大。

### 安装ImageMagick
更新安装源以确保安装成功
```sh
sudo apt-get update
```
```sh
sudo apt-get install imagemagick -y
```

### 压缩PNG
将**input.png**图片转换为**output.jpg**图片，且转换质量为50%
```sh
convert input.png -quality 50 output.jpg
```

![压缩前图像](input.png)

![压缩后图像](output.jpg)

![对比图像体积](compare_size.jpg)

通过对比压缩前后的图像体积，压缩后的图像大约**小了68倍**；此时分辨率不变，观感上模糊了一点，打开上述图片直接对比。**压缩JPG图像同理**。

{% cb 压缩GIF, false, false, false %}
{% note danger %}
暂时未找到有效的压缩办法，使用**gifsicle**压缩效果不理想，比如20MB的gif压缩后还有15MB左右
{% endnote %}
```sh
sudo apt-get install gifsicle
```
```sh
gifsicle -O3 --colors 256 input.gif -o output.gif
```
