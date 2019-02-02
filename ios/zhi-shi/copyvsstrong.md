---
title: 关于Copy和Strong关键词
date: '2017-11-05T22:03:32.000Z'
tags:
  - iOS
---

# 关于copy和strong关键词

昨天有人在群里提了个问题，关于浅拷贝和深拷贝的问题，我整理了整理，记录在这. 

## 问题

### NSArray 和 NSString 的 copy 操作为什么是浅拷贝？

我的理解是 NSArray 和 NSString 已经是不可变的了，那么完全没有必要 再开一个新的空间，也就是进行深拷贝。

### 而NSMutableArray 和 NSMutableString 的 copy 操作却是深拷贝？

这个原因也是和上面一样，如果这里也只是拷贝地址的话，那么NSMutableString改变了，被赋值的NSString也会改变。

```cpp
@interface ViewController ()
@property(nonatomic,strong )NSArray *Ary;
@end

@implementation ViewController
- (void)viewDidLoad {
    [super viewDidLoad];
    NSMutableArray *a= [NSMutableArray arrayWithObjects:@1,@2,@3, nil];;
    self.Ary=a;
    NSLog(@"%@ %p %p",_Ary,_Ary,a);
    [a addObject:@"4"];
    NSLog(@"%@ %p %p",_Ary,_Ary,a);
}
```

我们看到官网文档也说了，取决于类 ![C3DEFAE0-1A5A-4883-A9AD-0FE02B4B7957](https://img.wxz.name/C3DEFAE0-1A5A-4883-A9AD-0FE02B4B7957.png)

### 而可变对象和不可变对象的 mutable copy 也都是深拷贝

这里也是根据官网文档可以看到 ![9A8273D8-4A2A-46D1-8A47-C9456B9635EF](https://img.wxz.name/9A8273D8-4A2A-46D1-8A47-C9456B9635EF.png)

## 其他的探索

* 首先还要知道，对于容器类，深拷贝也只是单层深拷贝，如果要双层深拷贝，应该用归档再反归档

### 如果用copy和strong修饰符，分别实现的set方法

* 如果用copy 修饰符，对应的setter方法实现如下

```cpp
- (void)setArr1:(NSMutableArray *)arr1 {
    if (_arr1 != arr1) {
        [_arr1 release];
        _arr1 = [arr1 copy]; //内容拷贝，深拷贝
    }
}
```

* 如果修改成用strong修饰符，对应的setter方法实现如下

```cpp
- (void)setArr1:(NSMutableArray *)arr1 {
    if (_arr1 != arr1) {
        [_arr1 release];
        _arr1 = [arr1 retain]; //指针拷贝，浅拷贝
    }
}
```

> 值得注意的话，要用self.arry来复制，如果用下划线的方式会直接修改地址。

