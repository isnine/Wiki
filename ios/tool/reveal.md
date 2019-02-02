# 通过Reveal分析视图层级

因为某个侧滑功能希望能参照微信来做，但是手上又木有越狱机，所以这里主要通过IpaPatch+Reveal来进行分析。

## 准备工具

[Reveal](https://revealapp.com) [IpaPatch](https://github.com/Naituw/IPAPatch) [PP助手](https://www.25pp.com)

## 准备一个微信APP

首先需要强调的是，AppStore上下载的iPA是加密过的，所以这里我们通过[PP助手](https://www.25pp.com)来下载已经砸壳过后的IPA文件。 ![](https://img.wxz.name/15159380872900.jpg)

## 克隆好IpaPatch工程

IpaPatch是一个 免越狱调试、修改第三方App 的工具，主要过程分为这几部 1. 解压 IPA 文件 2. 用 IPA 文件的内容，替换掉 Xcode 生成的 .app 的内容 3. 通过 OPTOOL，将你代码生成的 Framework 及其他外部 Framework，注入到二进制文件中 4. 对这些文件进行重新签名

## 配置IpaPatch工程

1.在我们克隆完毕后，我们将我们在pp助手上下载好的微信.ipa重命名为app.ipa然后复制到工程的IPAPatch/Assets/中替换掉原来的app.ipa ![](https://img.wxz.name/15159382651784.jpg)

2.接着我们需要将Reveal中的framework文件复制到Assets/Frameworks/RevealServer.framework中 ![](https://img.wxz.name/15159384937355.jpg)

3.修改Display Name和Bundle Identifier。Display Name将加在原app名称之前，可与之前的app共存。 ![](https://img.wxz.name/15159385259266.jpg)

## 调试界面

1. 点击 Build 后就可以在我们真机上看到啦，然后打开Reveal，就可以看到微信的界面了

    ![](https://img.wxz.name/15159387058492.jpg)

## 开始分析

### 从层级开始分析

整体是一个tableview的界面，每个cell中，消息部分作为一个view与滑动部分的view所并列 ![Lark20180114221705](https://img.wxz.name/Lark20180114221705.png) 在一整个滑动view中，又是分为两个button，其中尺寸如第二条所分析 ![](https://img.wxz.name/15159396834913.jpg)

### 从尺寸分析

首先我的手机是iphone7 Plus,屏幕尺寸为 414  _736，在微信设置的滑动view的大小为 175_  68，其中

* 滑动view的大小为 175 \* 68
* “标为未读”按钮的宽度为116.6
* “删除”按钮的宽度为75.9
* 其中两个按钮中有重叠部分，应该是为了动画效果而保留的。

![](https://img.wxz.name/15159392238970.jpg)

### 从动画分析

#### 1. 刚开始滑动---&gt;滑动完成

开始滑动的那一刻立即创建两个button，同时两者随手势向左侧滑动。但是第二个“删除”按钮**滑动速度较慢**。 左侧聊天信息的view紧临“标为未读”的button。同时父视图cell位置不变 ![](https://img.wxz.name/15159398498752.jpg)

#### 2. 滑动完成，到达稳定的状态

cell的位置没变 但cell子视图的聊天view左移了，并且紧邻滑动view为同一层级 滑动view中又分为两个button，并且**位置有一定的重叠** ![](https://img.wxz.name/15159402686411.jpg)

#### 3. 返回过程

为之前动画的逆过程，不再赘述

#### 4. 长拖效果

拖动按钮时，按钮会随着一起拖动。 分析层级，发现滑动view的位置并没有感觉。 实际上**改变的是两个button的长度**，他们超过了父view，显示出了这样一个效果。 ![](https://img.wxz.name/15159405300287.jpg)

#### 5. 删除效果

**之前的“标为未读”按钮被删除了**，全部只剩下了“删除”按钮。动画过程，因为真机显示太快我还木有抓到。 ![2](https://img.wxz.name/2.png)

#### 6. 删除返回

随着手势的返回，将**“删除”按钮位置右移**，聊天view紧邻删除button，但主要的是**删除button的父view位置是不动的** ![](https://img.wxz.name/15159410586968.jpg)

### 总结

关于微信滑动的实现，几个关键的点 1. 首先层级是cell最上，包含聊天view和滑动view。其中滑动view在没有开始滑动时是不创建的，滑动开始立刻创建，收回后立即销毁。 2. 滑动动画上，像抽屉的效果是**通过移速不同**，“标为未读”在“删除”下面，这样拖动和拖回产生了一个折叠的效果 3. 自始至终，**cell的位置是不变得**，这也就保证了，无论的滑动view还是聊天view都不会因为父视图超过自身位置，因为响应链的原因而不响应。

