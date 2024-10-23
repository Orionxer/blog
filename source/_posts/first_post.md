---
title: My First Post
date: 2023-11-05 08:51:06
categories:
- Hexo
tags: [Hexo, Fluid, 博客]
banner_img: /images/first_post/bridge_wallpaperhub.jpg
index_img: /images/first_post/bridge_wallpaperhub.jpg
excerpt: 🎉摘要:本篇文章用于测试[Hexo]引擎+[Fluid]主题以及MarkDown语法
	
---
> 🎉摘要:本篇文章用于测试[Hexo]引擎+[Fluid]主题以及MarkDown语法

## ✨官方资源
### 🌊Hexo相关
[📌Hexo官网](https://hexo.io/zh-cn/)
[Fluid主题*Github仓库*](https://hexo.io/zh-cn/)
[Fluid主题*用户手册*](https://hexo.fluid-dev.com/docs/start/)
[Fluid主题*效果展示*](https://hexo.fluid-dev.com/posts/hello-fluid/)
[优雅的Fluid示例](https://hexo.fluid-dev.com/posts/fluid-write/index.html)
### 🍻免费图片共享
[Unsplash](https://unsplash.com/)
[Wallpaperhub](https://www.wallpaperhub.app/)
[Wallhaven](https://wallhaven.cc/)
### 😀Emoji在线查询
[webfx](https://www.webfx.com/tools/emoji-cheat-sheet/)

## 🎢Markdown标题级别
```md
# 一级标题
## 二级标题
### 三级标题
#### 四级标题
```

## 👔字体样式
斜体效果: *斜体*  
粗体效果: **粗体**
删除线效果: ~~删除线效果~~
H<sub>2</sub>O下标
Markdown<sup>TM</sup>上标

## 🛒引用
> 世上只有一种英雄主义，就是在认清生活真相之后依然热爱生活。--罗曼罗兰《米开朗基罗》1906
> 真正的勇士，敢于直面惨淡的人生，敢于正视淋漓的鲜血。--鲁迅《记念刘和珍君》1926

## 📋列表
### 无序列表
- item
	* item
	+ item
	+ ✅已完成待办列表
	+ ❎未完成代表列表
### 有序列表
1. item1
2. item2
3. item3
### 待办列表
- [ ] 未完成
- [x] 已完成

## 🎈图片
 ![Oscilloscope](/images/first_post/oscilloscope_unsplash.jpg)

## 💻代码效果
### 单行代码
Ubuntu获取可更新软件列表命令: `sudo apt update`; 执行升级命令`sudo apt upgrade`
### 代码块
```C
#include <stdio.h>
// 定义电池电量
static const uint8_t battery = 0x5A;
/**
 * @brief	经典Hello World
 */
int main(void)
{
	printf("Hello World\n");
	// 预期输出: The Battery is: 90%
	printf("The Battery is: %d%%\n", battery);
	return 0;
}
```

## 📊表格
借助Markdown在线表格生成工具以生成需要的表格代码
| 标题1 | 标题2 | 标题3 |
| ----- | ----- | ----- |
| 内容1 | 内容2 | 内容3 |
| 内容4 | 内容5 | 内容6 |

## 📚脚注

人类的悲欢并不相同，我只觉得他们吵闹。[^1]

提高学习的能力，比学到一门知识更重要。[^2]

[^1]: 鲁迅
[^2]: [Google](https://google.com)

## 🧪Tag模块(Fluid)
### 可选便签
{% note primary %}
**primary** *首要* 样式
{% endnote %}
{% note secnodary %}
**secnodary** *次要* 样式
{% endnote %}
{% note success %}
**success** *成功* 样式
{% endnote %}
{% note info %}
**info** *普通* 样式
{% endnote %}
{% note light %}
**light** *轻量* 样式
{% endnote %}
{% note warning %}
**warning** *警告* 样式
{% endnote %}
{% note danger %}
**danger** *危险* 样式
{% endnote %}

### 折叠块
{% fold warning @折叠块(点击展开/折叠) %}
🎉最新分支已支持折叠块功能，不需要切换develop分支
{% endfold %}

{% fold info @Fluid支持fold折叠块功能的操作指引 %}
```bash
# 修改博客根目录下的_config.yml的主题配置
theme: hexo-theme-fluid
# 进入博客根目录下的theme文件夹
cd theme
# git克隆仓库
git clone https://github.com/fluid-dev/hexo-theme-fluid
# 切换develop分支
git checkout -b develop origin/develop
# hexo清空并重启
hexo clean && hexo g
```
{% endfold %}

### ✅复选框
{% cb 可以选中/取消, true, false, false %}
```MD
{% cb 可以选中/取消, true, false, false %}
```
{% note danger %}
复选框最后一个参数false代表可以选中，属于bug，等待修复
{% endnote %}
{% cb 不可选中 %}
{% cb 普通示例, true, false, true %}
{% cb 内联示例, false, true %} 后面文字不换行
{% cb false %} 也可以只传入一个参数，文字写在后边（这样不支持外联）
## ⚖公式展示
$$
E=mc^2
$$
$$
\Gamma(z) = \int_0^\infty t^{z-1}e^{-t}dt\,.
$$

## 🎞组图
{% note warning %}
**组图**概念属于Fluid主题特色功能，仅支持`/images`路径下访问，不兼容`Hexo`文章资源文件夹的访问。
`Front-matter`中的`banner_img`属性和`index_img`属性同理。
{% endnote %}
{% gi total n1-n2-... %}
  ![](/images/first_post/bridge_wallpaperhub.jpg)
  ![](/images/three_body_unsplash.jpg)
  ![](/images/first_post/markus-spiske-gcgves5H_Ac-unsplash.jpg)
  ![](/images/first_post/oscilloscope_unsplash.jpg)
  ![](/images/first_post/spacex-PIOgkhaF3WA-unsplash.jpg)
{% endgi %}

## 🌇结束
