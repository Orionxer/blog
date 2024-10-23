---
title: HTTP基础调试
date: 2024-02-23 01:04:05
categories:
- HTTP
tags: [HTTP]
banner_img: /images/http-basic.jpg
index_img: /images/http-basic.jpg
excerpt: 通过命令行以及HTTPie客户端调试HTTP接口
---

{% note success %}
使用命令行以及客户端调试HTTP接口。HTTP的报文协议= Start Line + Header + CRLF + Body。其中`Start Line`又叫*起始行*，在请求中是*请求行*；在响应中是*状态行*。为本篇通过HTTP请求高德API接口，实现IP定位以及天气查询。
{% endnote %}

## 1.准备
使用任何高德的WEB服务API均需要提供`key`，简单理解为密钥。如果在HTTP请求时没有提供合法的`key`值，则高德API接口会返回`INVALID_USER_KEY`报错提示。
点击[高德地图官网开发者后台](https://lbs.amap.com/dev/)尝试申请`key`。假设申请成功的`key`如下
```sh
47478abb752cf12e1a6c91ab0892cae0
```
根据高德服务调用量说明[^1], 个人普通开发者每日请求次数（或者叫配额）
- **5000次** IP定位
- **300000次** 天气查询(预报)
### 1.1 IP定位文档
根据高德的IP定位文档[^2]，HTTP的请求方式为`GET`，请求URL是
```sh
https://restapi.amap.com/v3/ip
```
在请求中提供`key`值。如果在HTTP请求中不提供IP地址参数，则取客户http之中的请求来进行定位，即实现自动定位。如果请求成功则会拿到**城市编码**的值，即`abcode`
{% note warning %}
IP定位请求只支持国内的IP地址，如果是非法IP或者国外IP则其返回的结果中`province`和`city`属性均会返回空，也就是没有值
{% endnote %}
### 1.2 天气查询文档
根据高德的天气查询文档[^3]，HTTP的请求方式为`GET`，请求URL是
```sh
https://restapi.amap.com/v3/weather/weatherInfo
```
在请求中提供`key`以及IP定位查询到的`abcode`的值。注意：`abcode`的值应该赋值给`city`的键名。请求成功后则获取到当前城市的天气信息。
## 2.命令行
### 2.1 CURL命令行
一般Linux系统默认安装了`curl`，直接开始测试，以下的命令包含`&& echo`仅为了换行显示，对HTTP请求没有影响

#### 2.1.1 请求定位
```sh
curl https://restapi.amap.com/v3/ip -d key=47478abb752cf12e1a6c91ab0892cae0 && echo
```
![curl命令行请求定位](curl_ip.png)
{% note info %}
根据IP定位请求结果，城市编码`abcode`的值为**440100**
{% endnote %}
#### 2.2.2 查询天气
```sh
curl https://restapi.amap.com/v3/weather/weatherInfo -d key=47478abb752cf12e1a6c91ab0892cae0 -d city=440100 && echo
```
![curl命令行查询天气](curl_weather.png)
### 2.2 HTTPie命令行
#### 2.2.1 安装
```sh
sudo apt install httpie -y
```
#### 2.2.2 请求定位
```sh
http https://restapi.amap.com/v3/ip key==47478abb752cf12e1a6c91ab0892cae0
```
![httpie命令行IP定位](httpie_ip.png)
#### 2.2.2 查询天气
```sh
http https://restapi.amap.com/v3/weather/weatherInfo key==47478abb752cf12e1a6c91ab0892cae0 city==440100
```
![httpie命令行查询天气](httpie_weather.png)
## 3.HTTPie
⭐HTTPie[^4]客户端可以非常优雅地调试HTTP，完全免费，不仅能够兼容各个平台，而且能够**跨平台同步HTTP工作区**，使用github账户可以快速登陆。
### 3.1 网页客户端
#### 3.1.1 IP定位
1. 左侧边栏空白区域右键点击`Draft request`
2. 重命名`untitled`为`AMap IP API`
3. 右侧顶部区域选择HTTP请求方法为`GET`
4. 在`URL`处粘贴需要请求的`URL`
5. 在`Params`区域中添加键值对，输入键名为`Key`，键值从高德后台复制`key`值
6. 点击`Send`按钮开始请求
7. 观察请求结果，最右侧显示HTTP响应报文，HTTP状态行，响应头以及响应体
  - 响应行 = HTTP版本号 + HTTP状态码 + HTTP状态码补充解释
  - 响应头可以点击展开查看具体的键值对
  - 响应体可以选择不同格式展示，`JSON`，`RAW`以及`XML`等

![HTTPie网页客户端IP定位](httpie_web_ip.png)

观察HTTP响应报文：状态行显示`200 OK`，说明本次HTTP请求成功，而且响应体里的JSON键值对`"status":"1"`以及`"info":"OK"`均说明请求的`URL`以及`Params`参数合法。但是却没有查询到具体的城市。出现该问题的原因是Httpie网页客户端是由美国的服务器主机提供，虽然客户端运行在本地的浏览器。但是客户端发起HTTP请求的时候，实际上是从美国服务器IP去访问高德的IP定位接口的。根据高德的IP定位文档，国外的IP返回空。使用HTTPie本地客户端可以解决这个问题。

终端查询`httpie.io`的IP以及属地信息命令
```sh
curl https://ipinfo.io/$(dig +short httpie.io) && echo
```
![终端查询httpie.io信息](httpie_ip_info.png)

根据查询信息可以看到`httpie.io`运行于美国的服务器主机。或者终端查询域名的解析到的IP地址
```sh
nslookup httpie.io
```
再复制该IP地址到IP查询的WEB服务端进行查询，一样可以判断出由美国主机提供的服务。

![网页查询httpie.io信息](web_httpie_ip_info.png)

![高德IP定位接口省份以及城市显示为空](amap_ip.png)

#### 3.1.2 天气查询
步骤与IP定位相同，更换`URL`以及添加`Params`参数`city:440100`

![HTTPie网页客户端天气查询](httpie_web_weather.png)

### 3.2 本地客户端
下载安装HTTPie本地客户端且登陆账号之后，本地客户端会自动同步网页客户端创建的HTTP工作区

#### 3.2.1 IP定位
![HTTPie本地客户端IP定位](httpie_local_ip.png)

#### 3.2.2 天气查询
![HTTPie本地客户端天气查询](httpie_local_weather.png)

## 4.分析Header
以HTTP响应头`Header`中的`Content-Length`为例，尝试分析HTTP请求。`Content-Length`用于表示响应体`Body`的大小，且**特指字节大小**，非字符大小。如果响应体中包含中文，以UTF-8编码的标准，一个中文字符占用3个Byte。以字符的方式统计可能会出现错误，但是以字节则不会。测试方法如下：

- 在HTTPie本地客户端的IP定位请求成功后，设置`RAW`方式显示`Body`信息，再点击`Copy body`按钮
- 打开[字符串转16进制在线工具](https://tool.hiofd.com/hex-convert-string-online/)，粘贴至输入框，点击`字符串转16进制`按钮，点击`复制结果`按钮
- 打开[在线字数统计](https://www.eteste.com/)，粘贴至输入框，在输入框下方会显示**字符数**，往下滑动看到右下角的计算器，将字符数除以2就能得到实际的Content-Length
- 展开HTTPie的Header，查看Content-Length键值对，对比刚刚计算的Content-Length是否一致

{% note info %}
字符串转16进制的时候，一个字节占用两个字符，因此需要除以2才能算出实际的字节数。比如第一个字符`{`对应的16进制是`7B`。
{% endnote%}

![计算IP定位的Content-Length](ip_length.gif)

![计算天气查询的Content-Length](weather_length.gif)

{% note warning %}
有一些字符串转16进制的在线工具可能会**自动删除空格**，会导致计算的Content-Length与实际的不一致，比如[BEJSON的字符串转16进制的工具](https://www.bejson.com/convert/ox2str/)就会出现这个问题，需要注意。
{% endnote%}

## 参考
[^1]: [高德服务调用量说明](https://console.amap.com/dev/flow/manage)
[^2]: [高德IP定位文档](https://lbs.amap.com/api/webservice/guide/api/ipconfig)
[^3]: [高德天气查询文档](https://lbs.amap.com/api/webservice/guide/api/weatherinfo)
[^4]: [⭐HTTPie客户端官网](https://httpie.io/app)