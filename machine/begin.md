# easy技术文档
整体app采用swift开发，因初赛时限问题，队员们主要精力放在了app的核心模块 - 图片分类部分。
核心思路
- 通过爬虫抓取分类照片
- 通过caffe、tensorflow等框架，将分类图片训练成模型
- 对模型修改参数，不断调整
- 将模型通过coremltools等工具转换成apple支持的CoreML模型格式
- 通过模型对用户相册进行分类

## 通过爬虫抓取分类照片

## 第一步，首先我们会调用python脚本，借助百度的图片搜索，去搜索指定分类的图片，并下载在本地
![](https://img.wxz.name/15275212380763.jpg)

## 第二步，接下来我们通过tensorflow将分类的图片训练成模型
![](https://img.wxz.name/15275215012038.jpg)
其中optimized_graph.pb文件为我们输出的训练模型
retrained_labels.txt文件为我们输出的识别类型

## 第三步，因为iOS平台不支持JPEG decoder 所以将其移除
方案一 我们参照了tf-coreml上面的指南通过脚本去除
https://github.com/tf-coreml/tf-coreml/blob/master/examples/inception_v3.ipynb 
方案二 我们参照了tensorflow中的strip_unused命令来去除

我们可以看到去除后的模型，两者的在输入节点上的变化
![](https://img.wxz.name/15275222235387.jpg)

## 第四步，我们需要将模型运行在iOS平台上
方案一  通过tensorflow依赖库形式 直接运行tensorflow模型
1. 我们参照了官网demo https://github.com/tensorflow/tensorflow/tree/master/tensorflow/examples/ios
![IMG_8980](https://img.wxz.name/IMG_8980.PNG)
方案二  将tensorflow模型转换成apple的coreML模型
依据步骤https://github.com/tf-coreml/tf-coreml/blob/master/examples/inception_v3.ipynb 


