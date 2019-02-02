---
title: ipa重签名踩坑记
date: '2017-05-28T23:34:58.000Z'
categories:
  - iOS
---

# 对app进行重签名

前几天看到叉叉助手推了一个新功能，不越狱就可以用脚本挂游戏。当时黑人问号脸，这是什么黑科技。 后面点进去发现是将游戏ipa里面植入游戏脚本，然后用企业证书分发给用户使用。然而每个月要收30元，美其名曰证书费。 我突发奇想，我能不能用我的个人证书签名呢. 

## 原理

首先需要知道，我们在真机调试时，用到了一个Provisioning Profile供应配置文件，这个文件里面包含了app id，开发者信息，设备udid 三样东西。 app id - 用来识别将要安装的app udid - 用来识别设备（个人证书上限100个，企业无限） 开发者信息 - 自己电脑用CSR文件请求后得到的证书 ![](https://img.wxz.name/14959875852378.jpg) 当app安装到真机上，首先会确认这三个信息，是否都匹配，如果匹配则安装上去。 而我们也一样可以将别人的ipa，通过重签名的方式，安装到自己的手机上，唯一要改动的就是app id。

## 开始行动 - 命令行方式

首先我一开始采用的方式是通过命令行的方式 先通过PP助手的越狱商店下载ipa，之所以不通过itunes下载，是因为itunes商店的app都加了壳，我们需要自己砸壳太麻烦了。 \(1\)解压qq.ipa 找到Payload文件

```text
unzip qq.ipa //命令行解压
```

\(2\)将Payload目录中的\_CodeSignature文件删除

```text
rm -rf Payload/*.app/_CodeSignature/
```

\(3）将自己app打包导出ipa文件 解压后找到 embedded.mobileprovision 文件 并替换qq.ipa中的embedded.mobileprovision 文件

```text
cp embedded.mobileprovision Payload/*.app/embedded.mobileprovision
```

（4\)重新签名，“iPhone Distribution: XXXXXX”这个指的是自己的embedded.mobileprovision文件用到的签名证书名称，在xcode或钥匙串中可以找到

```text
/usr/bin/codesign -f -s "iPhone Distribution: XXXXXX" --resource-rules Payload/*.app/ResourceRules.plist Payload/*.app/
```

这里是第一个坑，在证书名称这里一直错误，后来知道可以使用

```text
 xcrun security find-identity -v -p codesigning
```

列举出所有的证书，然后复制进去 ![](https://img.wxz.name/14959883114267.jpg)

然后第二个坑，会发现提示 --resource-rules has been deprecated in mac os x &gt;= 10.10 上网搜索后，发现在[stackoverflow](https://stackoverflow.com/questions/26459911/resource-rules-has-been-deprecated-in-mac-os-x-10-10)这里有讨论，但是我没有解决，于是放弃了这个方法。 \(5\)重新打包

```text
zip -r qq.ipa Payload
/rm -rf Payload/
```

## 另寻新法 - iOS APP Signer

在命令行方式失败后，我开始寻找其他方法，于是发现了[iOS APP Signer](http://dantheman827.github.io/ios-app-signer/)这个方法。

![](https://img.wxz.name/14961507932167.jpg)

首先看这张图，主要要填的地方只有三个.

* 【1】为你要重签名的ipa文件，请注意这里要求是没加壳的，直接从appstore上面下载的是不行的，建议在PP助手上下载下来。
* 【2】为你自己的开发者证书。
* 【3】为供应配置文件。

这三个是必选项，剩下下面的地方分别是修改Bundle id，app名称，app版本。 我依照流程去做，结果每次都出错。直到最后看到了这个[这篇文章](http://www.hangge.com/blog/cache/detail_1219.html)最底下，然后用Team Provisioning Profile文件签名成功。 不得不说Team Provisioning Profile简直是神器，对任意bundle id皆有用。

## 签名完毕

打开Xcode-Window-Device。 点这个加号，选择刚刚的ipa安装到我们的手机上。 ![](https://img.wxz.name/14961513954999.jpg)

