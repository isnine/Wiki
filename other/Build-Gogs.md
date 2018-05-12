---
title: 搭建Gogs
date: 2017-01-09 21:56:34
categories: [Web]
---
# 引言
之前写了些实验室的建议，结果今天老师找到我要我负责下我建议上面内容的实施
大概是利用学校内网服务器的高速，搭建Git，搭建各组论坛，搭建FTP空间
感觉自己挖了一个坑给自己跳，不管怎么样先做吧
# 选择服务器
因为国内的VPS自己用来搭建博客了，所以在DigitalOcean新建了一个VPS
架着VPS配置服务器测试
服务器的话我直接选择Ubuntu14，毕竟熟悉点，服务器这种东西，感觉版本越高坑越多
# 配置基本环节
首先两条命令，更新包
``` 
sudo apt-get update
sudo apt-get upgrade
```
新建一个Git用户，如果直接用root用户建的话坑多
```
sudo adduser git 
su git//切换到git用户
cd ~  //进入用户git根目录
```
<!--more-->
# 安装Git
```
sudo apt-get install git //安装git
git --version //检查git是否安装成功
```
>这里如果提示无权限，或者不能使用sudo命令的话
编辑/etc/sudoers文件 在root ALL=(ALL:ALL) ALL 这行下添加git ALL=(ALL:ALL) ALL

# 安装Mysql数据库
```
sudo apt-get install mysql-server //安装mysql数据库    账户：root  密码：********
mysql --version //检查mysql版本判断是否安装成功
mysql -u root -p
mysql> SET GLOBAL storage_engine = 'InnoDB';
mysql> CREATE DATABASE gogs CHARACTER SET utf8 COLLATE utf8_bin;
mysql> GRANT ALL PRIVILEGES ON gogs.* TO ‘root’@‘localhost’ IDENTIFIED BY ‘itadmin’;
mysql> FLUSH PRIVILEGES;
mysql> QUIT；
```
# 安装golang环境
``` 
sudo mkdir goapp //go应用安装目录
sudo wget https://storage.googleapis.com/golang/go1.7.4.linux-amd64.tar.gz
tar -xzvf go1.7.4.linux-amd64.tar.gz -C /var/opt/
```
然后可以在/var/opt/的目录下发现一个go文件夹，这里包含了golang环境文件

配置golang环境
``` 
echo export GOROOT=/var/opt/go >> .bashrc
echo export GOBIN=$GOROOT/bin >> .bashrc
echo export GOARCH=amd64 >> .bashrc
echo export GOOS=linux >> .bashrc
echo export GOPATH=/home/gogs/goapp >> .bashrc
echo export PATH=.:$PATH:$GOBIN >> .bashrc
```
使用环境生效
```
source  .bashrc
```
# 安装gogs
``` 
sudo mkdir repositories//创建仓库目录
cd goapp 
sudo wget https://dl.gogs.io/gogs_v0.9.113_linux_amd64.tar.gz //下载gogs
tar -xzvf  gogs_v0.9.113_linux_amd64.tar.gz //解压
ls // 查看/home/git/goapp目录下文件和文件夹
cd gogs //进入解压创建的文件gogs
mkdir custom
mkdir custom/conf //创建自定义配置文件目录
sudo chmod -R 777 custom //修改custom文件夹权限
mkdir log  //创建日志目录
sudo chmod -R 777 log//修改log文件夹权限
```
# 启动gogs
```
cd /home/git/goapp/gogs
./gogs web
```
访问http://IP:3000/install来进行安装

# 安装
在mysql那主机地址我填了连不上去，所以直接用sqlite3做了

# 相关地址
[Gogs官网](https://gogs.io)

以上操作亲测成功，如果有遇到问题可以留言


