---
title: 关于搭建ShadowSocks以及使用
date: 2016-11-06 21:36:34
categories: [Other]
---
# 引言
因为得到了Github学生包送的50美金VPS的代金券，想着没处用，就用来当翻墙用
<!--more-->
# 服务器购买与配置
## 获得优惠
打开[digitalocean](https://www.digitalocean.com)，注册一个账户。
注册的时候需要填写信用卡，如果没有的话需要使用PayPal，注册一个就好，然后充值5美元
登录进去后在设置里面输入Github学生优惠包里面给的优惠玛，即可得到50美元
![](https://img.wxz.name/14784408449159.jpg)
这张图往下面拖一点，就是输入优惠码的地方了


## 购买服务器
点击页面右上角这个按钮，新建一个VPS
![](https://img.wxz.name/14784409805200.jpg)

系统建议Ubuntu 14.04，最低配置5美元每月即可
![](https://img.wxz.name/14784410566665.jpg)

地区选择，这里很关键，只推荐这两个地方,Ping大概在300ms左右，我使用的是新加坡的服务器，虽然比旧金山慢一点，但是晚上丢包率低一点。
![](https://img.wxz.name/14784411229022.jpg)

然后新建一个SSH Key 点击下面即可创建
![](https://img.wxz.name/14784412067193.jpg)

## 配置服务器
通过SSH连接到服务
先更新软件源
``` 
sudo apt-get update
```
安装Pip环境
```
sudo apt-get install python-pip
```
直接安装shadowsocks
```
sudo pip install shadowsocks
```

创建一个配置文件/etc/shadowsocks.json
```
{
    "server":"你的服务器IP",
    "server_port":8388,
    "local_address": "127.0.0.1",
    "local_port":1080,
    "password":"设置的密码",
    "timeout":300,
    "method":"aes-256-cfb"
}
```
> 可别直接复制，里面的中文改成你自己的东西

输入命令运行服务
```
sudo ssserver -p 8388 -k password -m aes-256-cfb -d start
```

# 使用Shadowsocks
## Windows环境
下载ShadowsocksR，如果是Let's try开发社社员，可以直接在群文件中下载
![群文件](https://img.wxz.name/14784894677606.jpg)


解压后打开Shadowsock
![](https://img.wxz.name/14784895009050.jpg)
![解压](https://cloud.smartisan.com/apps/note/notesimage/Notes_1477305122426.png)

依图进行设置,服务IP和密码换成自己设置的。如果是Let's try社员可以直接在通知群里面拿到社团翻墙用服务器。
![](https://img.wxz.name/14784887038051.jpg)

接着打开PAC模式(国内的网站还是用自己网，国外的网站走服务器代理)
![](https://img.wxz.name/14784895095942.jpg)

## Android环境
下载Shadowsock的Android软件，Let's try社员可以在群文件中下载
![](https://img.wxz.name/14784992391995.jpg)

打开APP，服务器和密码设置成自己的，然后点击连接即可。
![](https://img.wxz.name/14784992848256.jpg)

## IOS端环境
在AppStore中下载Wingy，进行服务器地址和密码的配置
![](https://img.wxz.name/14784997190986.jpg)


