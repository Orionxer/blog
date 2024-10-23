---
title: 开源协议
date: 2023-12-13 00:31:07
categories:
- Other
tags: [License]
banner_img: /images/open_source.png
index_img: /images/open_source.png
excerpt: 了解主流的开源协议以及如何使用开源协议
---

## 开源背景
开源**OpenSource**，目的是为了促进知识和技术的共享/创新。开源协议也叫开源许可。开源软件是一种技术和立场中立的使用许可证约束的开放源代码的软件，以开放协作方式进行开发，其源代码允许任何人使用、拷贝、修改以及重新发布。[^1]
### 知识产权

<style>
.center 
{
  width: auto;
  display: table;
  margin-left: auto;
  margin-right: auto;
}
</style>
<div class="center">

|      著作权         |          著左权        |     公共领域       |
| :-------------------: | :---------------------: | :------------------: |
| Copyright           | Copyleft              | Public Domain      |
| All Rights Reserved | Mandatory Open Source | No Rights Reserved |
</div>

#### 著作权
著作权**Copyright**指作者对其创作作品享有的独占权利，保留所有权利，即**All Rights Reserved**。虽然保护了作者的权利但是并不利于知识的共享。
#### 著左权
著左权**Copyleft**，主张通过许可证开放作品的使用和传播，要求保持作品的开放性，禁止闭源。
#### 公共领域
公共领域**Public Domain**的概念完全与著作权相反，不保留任何权利，即**No Rights Reserved**。任何进入了公共领域的作品不再有任何的限制，完全自由。如果作品遵守了`CC0`协议或`Unlicense`协议，则该作品进入了公共领域。

### Create Commons
知识共享许可协议**Create Commons**是一种公共著作权许可协议(简称**CC协议**)，其允许按照多种需求分发受著作权保护的作品[^2]。目前主流的CC协议是**CC 4.0**，包含了四项基本权利: 
<div class="center">

| 基本权利             | 意义         |
| ------------------  | -------------- |
| `BY` Attribution    | 署名         |
| `SA` ShareAlike     | 以相同方式共享 |
| `NC` Non-Commercial | 非商业性使用 |
| `ND` No Derivative  | 禁止演绎   |
</div>

CC协议的作用范围比开源协议更广(*开源主要适用于软件和代码*)，可以用于各种创作作品，比如适用于文章博客，硬件电路，3D结构设计等，比如本篇文章使用的就是[CC BY-NC 4.0 协议](https://creativecommons.org/licenses/by-nc/4.0/deed.zh-hans)，可以单击定位到[文章底部](#参考链接)查看。

{% note info %}
CC协议中最基础的要求是**署名**，如果作品的作者放弃署名，则该作品使用的是**CC0 1.0协议**[^3]，作品默认进入公共领域。
{% endnote %}

### 开源≠免费
其实就是开源软件和免费软件的概念。混淆的原因是用户在下载使用开源软件以及免费软件的过程中均没有付费的行为。仅从用户使用角度来看，确实没什么区别，好用就行。
对于开源软件而言，用户在遵守了对应的开源协议后，可以进行二次修改并分发。而免费软件只是一种商业的行为/策略，一般不会允许用户进行修改并分发。
{% note success %}
如果是个人学习或使用，不管是下载还是二次开发，只要不涉及**分发**或**商业**行为的话，基本所有声明了CC协议或开源协议都是允许的，无需关注具体的协议条款。
{% endnote %}

## 主流的开源协议
分为两大类：宽松协议(**Permissive**)和严格协议(**Copyleft**)。
<div class="center">

|      宽松协议        |          严格协议     |
| :-------------------: | :---------------------: |
| Permissive           | Copyleft              |
| `MIT`, `BSD`, `Apache` | `AGPL`, `GPL`, `LGPL`, `MPL`, `EPL`, `CDDL` |
| 允许闭源/商用 | 不允许闭源，适当允许商用 |
</div>

![主流开源许可证对比[^4]](1702542586611.jpg)
### MIT协议
**MIT**协议是主流的协议中最宽松的协议，分发时只要求**署名作者**，当然也不承担代码使用的风险。
![MIT License](mit_license_logo_by_excaliburzero_d9ur2lg-414w-2x.jpg)
### BSD协议
一般BSD协议指的是**BSD 3-Clause**协议，其典型的要求是：**不允许使用源项目进行推广/宣传**。
![BSD License](1024px-License_icon-bsd-88x31.svg.png)
### Apache协议
一般Apache协议指的是**Apache 2.0**协议。其典型要求是：**需要对修改的代码进行说明**，且附带原来代码的协议。
![Apache License](1702543802916.jpg)
### GNU GPL系列协议
#### GPL协议
一般GPL协议指的是`GPL 3.0`协议，不过`GPL 2.0`协议是最广泛的的GPL版本。比如`Linux`和`git`遵守的就是`GPL 2.0`协议。
GPL协议允许用户自由复制，修改，发布。**商业软件不允许使用GPL协议的代码**。GPL协议是允许用户收费的，只是用户需要在收费的时候说明收费原因以及分发GPL的副本给客户，明确告知客户可以免费获取该开源软件。如果项目中使用了关于GPL的代码，那么该项目也必须按照GPL协议进行发布，也就是GPL感染，目的是为了强力促进开源共享。
![GPL License](GPLv3_Logo.png)
#### LGPL协议
一般LGPL协议指的是`LGPL 3.0`协议(旧版本是`LGPL 2.1`协议)。**商业软件可以使用LGPL的代码，但是不能修改源代码。**如果项目通过动态链接的方式调用LGPL协议的项目，则该项目调用接口的部分必须开源，其他部分允许闭源。
![LGPL License](LGPL.png)
{% note info %}
一个项目中可能包含多个开源协议，比如**FFmpeg** 基本包含了**GPL**系列全家桶。
但是不同协议不一定能兼容，比如项目中同时包含了**Apache-2.0**以及**GPL-2.0**协议，则该项目的开源协议声明是无效的。
{% endnote %}
## 如何使用开源协议
以github工程为例，在项目的根目录中创建`LICENSE`文件，并写入对应的开源协议。如果该文件包含的标准的主流协议，则github会自动识别协议。
![github自动识别项目协议](image.png)
<!-- ![What License to Choose[^3]](cover.webp) -->
具体的开源协议内容推荐以下两种获取方式，以`MIT`协议作为例子
### OSI官网复制
#### 查找协议
打开[OpenSource Initiative](https://opensource.org/license/)，在`Search License`处输入`MIT`,搜索后点击`The MIT License`
![搜索协议](osi_mit_search.png)
#### 复制修改
复制整个协议，粘贴至`LICENSE`文件中，并修改`<Year>`和`<COPYRIGHT HOLDER>`
![复制协议](osi_mit_copy.png)
假设年份是`2023`年，作者是`Orionxer`，则MIT协议如下
{% fold info @MIT协议示例 %}
```text
Copyright <2023> <Orionxer>

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the “Software”), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED “AS IS”, WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
```
{% endfold %}

### Choose a License插件
{% note success %}
⭐推荐使用VSCode的**Choose a License**插件自动为项目生成开源协议，非常优雅。
{% endnote %}
#### 1.安装插件
![安装Choose a License插件](license_plugin.png)
#### 2.设置年份和作者
`Ctrl + Shift + P`调出VSCode控制台，输入`License: set`，根据提示选择设置年份以及作者。该设置是一次性的，后续生成任意的协议会根据预设好的年份以及作者自动替换。
![设置年份和作者](license_set.png)
#### 3.生成协议
`Ctrl + Shift + P`调出VSCode控制台，输入`License:`，选择`License: Choose license`。输入`mit`，点击`MIT License`就能生成协议(*如果项目中已经存在了协议，则该插件会询问是否替换协议*),点击`LICENSE`文件查看协议是否正常生成。
![MIT协议生成成功](license_generate.png)
{% note warning %}
点击生成协议前需要确保VSCode的工作区已经处于项目的根目录，否则该插件会在错误的位置生成协议。
{% endnote %}
## 参考链接
[^1]: [⭐开源指北：一份给开源新手的保姆级开源百科](https://oschina.gitee.io/opensource-guide/guide/)
[^2]: [知识共享许可协议](https://zh.wikipedia.org/wiki/%E7%9F%A5%E8%AF%86%E5%85%B1%E4%BA%AB%E8%AE%B8%E5%8F%AF%E5%8D%8F%E8%AE%AE)
[^3]: [CC0 1.0协议](https://creativecommons.org/publicdomain/zero/1.0/deed.zh-hans)
[^4]: [如何选择开源许可证-阮一峰](https://www.ruanyifeng.com/blog/2011/05/how_to_choose_free_software_licenses.html)

