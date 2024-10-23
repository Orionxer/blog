---
title: 配置SSH免密码登录
date: 2023-12-11 20:07:11
categories:
- Linux
tags: [Linux, SSH]
banner_img: /images/ssh_psk.jpg
index_img: /images/ssh_psk.jpg
excerpt: 在Windows/Linux/VSCode下配置SSH免密登录，提升SSH登录便捷性与安全性
---
> 在Windows/Linux/VSCode下配置SSH免密登录，提升SSH登录便捷性与安全性

## SSH密钥登录优势
### 🔒安全性
相对于传统的密码登录(*密码在网络中传输*)，密钥对登录因为其非对称加密的特性会极大提升安全性。`ssh-keygen`使用了非对称算法生成一个公钥和私钥(默认算法是`SHA256`)，私钥一直存储在客户端，而公钥保存至服务器。在SSH登录过程中不涉及密码传输以及私钥传输，即便是公钥被窃取，也无法逆向推导出私钥。
{% fold info @SSH登录的过程[^1] %}
1. 客户端通过`ssh-keygen`生成自己的公钥和私钥。
2. 手动将客户端的公钥放入远程服务器的指定位置(`~/.ssh`)。
3. 客户端向服务器发起 SSH 登录的请求。
4. 服务器收到用户 SSH 登录的请求，发送一些随机数据给用户，要求用户证明自己的身份。
5. 客户端收到服务器发来的数据，使用私钥对数据进行签名，然后再发还给服务器。
6. 服务器收到客户端发来的加密签名后，使用对应的公钥解密，然后跟原始数据比较。如果一致，就允许用户登录。
{% endfold %}
### 🚗便捷性
VSCode使用传统密码远程连接主机且打开工作区的时候，往往需要手动输入2-3次密码，长期使用则十分繁琐。使用密钥对登录则只要确保SSH密钥对配置成功，后续SSH登录都不再需要输入密码。
## SSH客户端配置
以服务器`unraid`, 登录用户`root` 为例
### Windows客户端
{% note primary %}
以用户名`Orionxer`为例，SSH公钥、私钥以及配置文件存储在该路径`C:\Users\Orionxer\.ssh`
{% endnote %}
{% note warning %}
不建议使用`PowerShell`进行SSH的密钥对生成以及配置，`PowerShell`缺少`ssh-copy-id`[^2]命令
{% endnote %}
#### 1.进入密钥路径 
打开`Git Bash`终端并进入`.ssh`文件夹
```Bash
cd ~/.ssh
```
#### 2.生成密钥对
私钥名称格式 = *Host_id_rsa*(有助于区分多个服务器),比如`unraid_id_rsa`,则该命令会自动生成公钥名称：`unraid_id_rsa.pub`
```Bash 
ssh-keygen.exe
```
![第一个参数输入私钥名称；第二、三个参数直接回车确认](ssh-keygen.jpg)

#### 3.传输公钥到服务器
```Bash
ssh-copy-id -i unraid_id_rsa.pub root@unraid
```
<!-- ![输入服务器密码](/images/ssh_psk/ssh-copy-id.jpg) -->
![输入服务器密码](ssh-copy-id.jpg)
#### 4.配置ssh_config
```Bash
vim config
```
添加以下配置
```yml
Host unraid
    HostName unraid
    User root
    PreferredAuthentications publickey
    PubkeyAuthentication yes
    IdentityFile "C:\Users\Orionxer\.ssh\unraid_id_rsa"
```
以上配置要求SSH登陆使用密钥对的验证方式，更多参数定义查阅SSH官网[^3]
#### 5.SSH登录
```Bash
ssh root@unraid
```
![Windows-SSH免密登录成功](windows-ssh-login.jpg)
### Linux客户端
{% note primary %}
一般的Linux系统，SSH公钥、私钥以及配置文件存储在该路径`~/.ssh`
*Linux的SSH密钥对生成与配置步骤与Windows一致。*
{% endnote %}
使用`WSL2 Ubuntu`系统，服务器`unraid`, 登录用户`root`为例
#### 1.进入密钥路径 
打开`Bash`终端并进入`.ssh`文件夹
```Bash
cd ~/.ssh
```
#### 2.生成密钥对
根据提示填写私钥名称以及回车确认密码，生成公钥以及私钥
```Bash
ssh-keygen
```
#### 3.传输公钥到服务器
根据提示输入服务器密码
```Bash
ssh-copy-id -i unraid_id_rsa.pub root@unraid
```
#### 4.配置ssh_config
```Bash
vim config
```
添加以下配置
```yml
Host unraid
    HostName unraid
    User root
    IdentityFile "~/.ssh/unraid_id_rsa"
    ForwardAgent yes
```
#### 5.SSH登录
```Bash
# 可以使用tab自动补全
ssh unraid
# 或者
ssh root@unraid
```
![Linux-SSH免密登录成功](linux-ssh-login.jpg)
### VSCode客户端
确保VSCode已经安装`Remote-SSH`插件, 且假设VSCode安装在Windows上
![VSCode-SSH](ssh-vscode.jpg)
## 其他传输公钥的方式
{% note info %}
本质上能够将公钥内容追加至服务器的`~/.ssh/authorized_keys`文件里都可以
{% endnote %}
以服务器`Tower`, 登录用户`root`为例
### SCP上传公钥
```Bash
# 客户端上传公钥
scp Tower_id_rsa.pub root@Tower:~/.ssh
# 服务器保存公钥至authorized_keys(需要远程登陆SSH服务器)
cat ~/.ssh/Tower_id_rsa.pub >> ~/.ssh/authorized_keys
```
### 直接复制公钥
以客户端上的公钥`Tower_id_rsa.pub`为例，其内容如下
```text
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDLm+9JG+yvnweKub7pKL4R/M7SeQ5976D1whSI28HbSZamdGRTZdQt/gmPYLA4gYiwgmHhEX2wNQA0DWbzktFy6raR8cwglKU3TTKAUFh4SUHaxHHl5zLnd47hvDHEEOQT3IF/zbTwvOQXcDYSqIX6BrvOMV7nacDaHUzj/+iN8i2m5C+A7rwfUzp3M+aZ6qQqZxlHAfBxWGKczj2OBvsKajSefDu1cbHeM9uF7CP4Fyl7LZcuQ9iM+xEvO8NWyQ+OUG45y1G1zw6WKzkyNmvqwe8yJn1J1lSABkD7jFdAAGY+0WvG/QcS4c8ZwZnqE8nDT5uxKJ5h7y1HbStEkDWBpagr19Rr7A2H7Xv1h4fZW4onRX/yFkoDRPY55fqh7ewIJGDk0ZB0CCfr6cHgMK7Cwq4Sc0owq5FpaRsgZQyD3PIyRtBG1lA7/Fun08FfWHLwN9eRzKEl5X1xp47alGWFcMWGpE5v/9GSEn8O8J4XGIDIFp0Ub+8xNDomOo8S4WE= Administrator@PC-20200903OFZO
```
复制以上内容后，追加到服务器的`~/.ssh/authorized_keys`中，该文件一行存储一个公钥，**注意不要覆盖到其他的公钥**
## 参考链接
[^1]: [网道-SSH密钥登录](https://wangdoc.com/ssh/key)
[^2]: [PowerShell无法使用ssh-copy-id命令](https://www.cnblogs.com/zhouzhihao/p/17087666.html)
[^3]: [ssh_config-Linux_man_page](https://linux.die.net/man/5/ssh_config)
