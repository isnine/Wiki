---
title: iOS UndoManager
date: 2018-05-02 20:55:17
categories: [iOS]
---
这几天在实现iOS撤销操作的坑，发现网上相关学习资料的是在是太少了，这里做一个文档的记录。


# UndoManager的介绍
>Overview
You register an undo operation by calling one of the methods described in Registering Undo Operations. You specify the name of the object that’s changing (or the owner of that object) and provide a closure, method, or invocation to revert its state.
After you register an undo operation, you can call undo() on the undo manager to revert to the state of the last undo operation. When undoing an action, UndoManager saves the operations you reverted to so that you can call redo() automatically.

从开发文档中看，它的目的简单的说就是，在你调用方法的时候，你可以注册一个undo操作。并且在你注册undo操作后，你可以执行undo()方法来撤销，同时在你撤销时，保存你的撤销操作，你可以调用redo() 来复原

# 实际开发中需要用到的几个重要方法
## canUndo()与canRedo()
通过这两个方法，我们可以知道当前是否可以调用Undo()和Redo()，在实际开发中，我们常常需要做一个undo和redo的按钮，那么通过这个方法，我们可以知道是否开启这个按钮
![](https://img.wxz.name/15252676183619.jpg)
``` 
    undoManager?.canUndo == true {
        // 开启撤销按钮
    } else {
        // 关闭撤销按钮
    }
```

## Undo()与redo()
调用这两个方法，可以执行相应的操作
``` 
    undoManager?.undo()
    undoManager?.redo()
```

## 注册
这是最关键的地方，这里以UITextView为例
```
func add(style: style, range: range) {
    let textView = UITextView()
    textView.undoManager.registerUndo(withTarget: self) { this in
                this.add(style: style, range: range)
    }
    // add方法的实现
}
```
这里以一个为文本添加样式的add方法为例子，我们每次在调用add方法时，都会先在undoManager中注册一个撤销的方法，要注意一定要先注册再执行方法，深坑



# 相关链接
[使用 NSUndoManager 来进行撤销和重做](http://swift.gg/2015/11/10/ios-undo-and-redo-with-nsundomanager/)
[NSUndo​Manager](http://nshipster.cn/nsundomanager/)



