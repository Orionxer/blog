---
title: Clash快速入门到精通
date: 2024-03-26 19:28:55
categories:
- Other
tags: [Clash]
banner_img: /images/clash.jpg
index_img: /images/clash.jpg
excerpt: 介绍如何优雅的使用Clash代理：包含自定义代理规则、局域网设备代理、虚拟网卡等
---

{% note success %}
代理也可以叫劫持，本质上就是域名解析，通过将请求转发到指定的服务器(通常是海外)来访问被大陆封禁的服务。比如浏览器访问谷歌网站，这是最常见的是应用层HTTP代理。Clash内置规则分流系统，能够自定义代理规则；局域网设备代理可以让同一局域网的设备实现代理而无需在每一台设备上安装Clash；虚拟网卡可以从物理层实现代理。
{% endnote %} 

- [ ] vscode markdown 预览gif
- [x] 文章设置可见列表
- [ ] 预览折叠块
- [x] 双向链接
- [ ] 添加Foam说明
- [ ] 优化知识图谱色彩
  
[[linux-cmd-1]]

## 1.代理规则
通过[Clash for Windows 代理工具使用说明](https://docs.gtk.pw/)快速了解Clash
### 1.1 代理(Proxies)
常见的代理方式有三种：
- 全局(Global)： 所有请求**都代理**
- 规则(Rule): 请求根据配置文件的规则进行**部分代理**，例如：国内网站不走代理，国外网站走代理
- 直连(Direct): 所有请求**都不代理**

![常见代理方式](proxy_way.png)

### 1.2 配置(Profiles)
日常使用最多的配置规则是`DOMAIN-SUFFIX`域名后缀匹配以及`DOMAIN-KEYWORD`域名关键字匹配

#### 1.2.1 DOMAIN-SUFFIX
域名后缀匹配的意思是请求中的**一级域名等于预设的域名**，则执行代理。比如指定域名`google.com`的配置为`DOMAIN-SUFFIX`：

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

|完整域名|一级域名|后缀匹配结果|是否代理|
| :--:| :--: |:--: |:--: |
|www.google.com| google.com|匹配|是|
|images.google.com| google.com|匹配|是|
|www.googleapis.com| googleapis.com|不匹配|否|
|www.google.com.hk| google.com.hk|不匹配|否|
</div>

#### 1.2.2 DOMAIN-KEYWORD
域名关键字匹配很简单，只要请求中的**域名中包含预设的域名**，则执行代理。比如指定`google`的配置为`DOMAIN-KEYWORD`

<div class="center">

|完整域名|关键字包含结果|是否代理|
| :--:| :--: |:--: |
|www.google.com| 包含|是|
|www.googleapis.com|包含|是|
|www.google.co.jp| 包含|是|
</div>

## 2.添加订阅
一般的机场会提供可以直接订阅的链接，复制到Clash导入。

![添加订阅](download_clash.png)

主界面开启**系统代理**，正常情况下浏览器就可以访问谷歌了

![开启系统代理](enable_proxy.png)

![浏览器访问谷歌](google_http.png)

## 3.自定义代理
{% note primary %}
Clash会包含基础的代理规则，也可以自行订阅代理规则，通常这些代理规则足够覆盖日常使用。如果某个域名没有被代理但是实际上被墙了，则临时开启**全局代理**是解决问题的最快方式。但最优解是将该域名加入自定义的代理规则，不然频繁切换代理策略会相当繁琐。
{% endnote %}

如果是在Clash的**配置**标签页修改某个域名的代理规则，虽然设置后能够有效代理，但是一旦**点击了更新的按钮，则代理规则会失效**。

![仅单次生效的代理规则](once_proxy.png)

真正永久有效修改代理规则需要在**设置**标签页的**预处理配置文件**中修改，点击编辑按钮后，Clash会打开编辑器，复制以下的示例配置并根据自己的实际情况修改，保存后退出，**重启Clash**。

![永久生效代理规则](eternal_proxy.png)

将域名`microsoft.com`、`office.com`、`office.net`以`DOMAIN-SUFFIX`的代理方式加入到`FreeGecko`的代理规则中，**将url替换成自己的订阅链接**，如果有需要添加的规则，继续往下追加即可(注意缩进，需要符合`yaml`语法)

```yml
parsers: # array
- url: http://subscribed_url
  yaml:
    prepend-rules:
      - DOMAIN-SUFFIX,microsoft.com,FreeGecko
      - DOMAIN-SUFFIX,office.com,FreeGecko
      - DOMAIN-SUFFIX,office.net,FreeGecko
```

![代理规则修改成功](proxy_success.png)

{% note info %}
如果在使用Microsoft的TODO List发现总是无法同步数据，查看Clash的日志就能发现在同步数据的期间访问了和`office.com`和`office.net`，这两个域名在不加入代理的情况下不一定能够解析成功。
{% endnote %}

## 4.局域网连接
Clash的**允许局域网连接**功能可以让处于同一个局域网下的所有设备实现代理，只需要在任意一台设备上部署好Clash并开放端口。以下以iPhone实现代理为例，需要确保iPhone与电脑处于同一局域网下：

- 打开Clash主界面，点击左侧**常规**选项卡
- 确保**允许局域网连接**处于启用状态且**端口**设置为`7890`
- 查询本机**内网IP地址**，假设查询到`192.168.100.38`
- iPhone进入无线局域网设置，点击已连接WiFi的ℹ符号
- 往下滑动点击**HTTP代理**
- 点击手动，**服务器**填`192.168.100.38`，**端口**填`7890`，点击右上角**存储**按钮
- 打开浏览器访问谷歌，iPhone内网代理成功

![启用局域网连接](enable_lan.png)

{% gi total n1-n2-... %}
  ![选择WiFi信息](/images/clash/wifi_info.png)
  ![选择HTTP代理](/images/clash/set_proxy.png)
  ![设置并保存内网代理地址和端口](/images/clash/proxy_ip_port.png)
  ![iPhone浏览器成功访问谷歌](/images/clash/google.png)
{% endgi %}

## 5.UWP网络限制
**解除UWP网络限制**一般用于解决微软商店无法访问的情况[^1]，其他服务以及应用同理，操作步骤如下：

- 打开Clash主界面，点击左侧**常规**选项卡
- 找到解除UWP网络限制，点击**启动助手**
- 往下滑动找到并**选中**`Microsoft Store`(可以顺便把微软相关的应用都选中)
- 点击**Save Changes**保存，重新打开微软商店

![解除微软商店的UWP网络限制](uwp_microsoft.png)

![成功访问微软商店](microsoft_success.png)

## 6.TUN模式

{% note primary %}
Clash提供了一个TUN模式，也就是虚拟网卡模式，能够让整个网络环境从物理层就实现代理，**网卡级代理**。比如可以本机可以PING通谷歌，或者WSL终端环境的代理。
{% endnote %}

TUN模式打开需要安装服务且Clash会自动重启，启动成功后，系统的网络接入方式就是Clash虚拟的网卡

![启动TUN模式](enable_tun.png)

<div class="center">

|代理类型|OSI网络层|网络名称|
| :--:| :--: |:--: |
|浏览器访问谷歌| 第七层|应用层|
|长城防火墙拦截| 第三层|网络层|
|虚拟网卡| 第一层|物理层|
</div>

前面提到的访问谷歌网站，浏览器使用HTTP方式访问谷歌，HTTP属于应用层协议；长城防火墙的拦截方式主要是污染DNS或者封禁IP，使域名无法解析到有效的IP或者使IP失效；而虚拟网卡是在第一层就实现了代理，也就是物理层。

简单来说，只开启Clash代理的情况下，可以访问谷歌，但是无法Ping通谷歌的域名；如果打开Clash的TUN模式，也就是通过虚拟网卡访问谷歌，也就是可以Ping通。以`google.com`域名为例，使用WSL执行PING命令，分别测试TUN模式关闭与打开的结果
```sh
ping google.com
```

![关闭TUN模式的PING结果](ping_tun_off.png)

![打开TUN模式的PING结果](ping_tun_on.png)

### 6.1 IP定位验证
根据[IP Geolocation](https://ip-api.com/docs/api:json)文档，客户端尝试请求该API接口`ip-api.com/json`，如果不指定IP，则该接口会根据客户端的IP地址进行定位并返回相关结果。

> If you don't supply a query the current IP address will be used.

WSL终端测试命令，分别测试TUN模式关闭与打开的结果
```sh
http ip-api.com/json
```

![关闭TUN模式的IP定位结果](ip_tun_off.png)

![打开TUN模式的IP定位结果](ip_tun_on.png)

### 6.2 TCP握手验证
{% note primary %}
当客户端发起请求时，客户端就与服务器建立连接。以TCP为例，在成功建立连接后，服务器可以获取客户端的公网IP(一般情况下，真实客户端并不拥有公网IP，而是与许多客户端共用一个公网IP)。服务器根据客户端的公网IP就能粗略判断客户端的IP地址。
{% endnote %}

测试过程如下(客户端与服务器仅建立了**传输层**的连接)
1. 服务器自行准备可以监听端口1884的服务，*理论上只要能启动基于TCP协议的应用层服务均可，比如HTTP或者MQTT*
2. 客户端使用`telnet`命令与服务器建立连接，例如
```sh
telnet gogo.uno 1884
```
3. 服务器使用`netstat`命令查询1884端口的连接信息(包含客户端的公网IP地址)
```sh
netstat -tn | grep 1884
```
4. 服务器查询客户端IP地址信息，根据实际情况替换IP地址
```sh
http ip-api.com/json/113.109.251.116
```

![关闭TUN模式的查询客户端IP地址信息](cleint_ip_tun_off.png)

![打开TUN模式的查询客户端IP地址信息](cleint_ip_tun_on.png)

也可以使用`netstat`命令查询连接，提取出IP地址后作为参数传递给IP定位接口
```sh
http ip-api.com/json/$(netstat -tn | grep ':1884' | awk '{print $5}' | cut -d: -f1 | sort | uniq)
```

![服务器组合命令查询客户端IP地址信息](ip_combine.png)

## 参考

[^1]: [Microsoft Store打不开，最好的解决办法](https://zhuanlan.zhihu.com/p/343342776)


