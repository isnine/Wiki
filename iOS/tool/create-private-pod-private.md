---
title: 创建自己的CocoaPods库
date: 2018-04-30 18:11:12
categories: [iOS]
---
一直想自己折腾一个第三方库，但是苦于之前一直没写啥好的控件
最近工作上写了几个ui控件，终于满足了下开源的心愿，这里记录下步骤
<!--more-->
# 本地建一个库
## 创建
```
    pod lib create xxxx
```
![](https://img.wxz.name/15250836204894.jpg)

xxxx为库的名字，然后会有一系列的选项，根据自己的需求进行选择
## 打开新建的模板库，将自己的代码放进去
![](https://img.wxz.name/15250835729749.jpg)
## 对库进行基本的设置 
打开xxxxx.podspec
像s.homepage,s.source等配置项改成自己准备传的仓库地址
# 将库传到远程库上
```
    git add .
    git commit -m "xxx"
    git remote add origin xxxxx
    git push master origin -u
    git tag 1.0.0  // 这里改成库的版本
    git push origin 1.0.0
```
这样库的代码就传到远程上了，如果是不希望给人看到的，就放在私有库上，反之就公开
# 提交到官方索引库
## 注册一个账号，比较坑的是如果换电脑需要再注册一次
```
    pod trunk register 邮箱 '名字' --description='描述' --verbose
```
然后打开邮箱，去收注册邮件
## 将自己的库的文件push上去
```
    pod trunk push 
```
## 测试
```
    pod search xxxxx  //xxx是库的名字
```
# 创建一个自己的私有库
我们有时不希望这个库被所有人可以用，比如公司等场景，那么我们需要创建一个自己的私有库
## 新建一个私有库
首先创建一个这样的仓库，这个私有库应该是公网上的私有仓库，或者公司内网上的仓库，接着将地址加进来
![](https://img.wxz.name/15250848614083.jpg)
```
    pod repo add MyRepo xxxxxxxx.git
```
## 推送库到私有库中
然后我们将之前的库推送到这个私有库中
```
     pod repo push MyRepo XXXXXX.podspec --allow-warnings
```
## 使用私有库
我们在podfile文件中，新加上
```
    source 'xxxxxxx'  
```
xxxx为我们私有库的地址，这样就完成了


# 参考文章
[私有库的创建](https://www.jianshu.com/p/0c640821b36f)
[公有库的创建](http://qiubaiying.top/2017/03/08/CocoaPods公有仓库的创建/)
[cocoaPods的学习和使用](http://luoxianming.cn/2016/03/27/CocoaPods/)


