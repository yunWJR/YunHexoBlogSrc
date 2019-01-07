---
title: UIViewController 生命周期方法与业务逻辑
date: 2018-04-18 18:45:04
categories: iOS
tags: [iOS,UIKit]
---

# UIViewController 生命周期方法与业务逻辑

**UIViewController 生命周期的主要包括9个方法：**

![](http://qnyunyun.yunsoho.cn/VcLife.png?imageMogr2/thumbnail/!70p)


## init

> 默认初始化

- **业务逻辑**

    可以初始化数据

- **注意事项**

    不要创建 view。不要调用 self.view。因为view是lazyinit的，调用 self.view，会导致viewcontroller创建view。
    
## loadView

> 加载 view，如果是从 nib 文件或者 stroyboard加载，则加载相关文件，如果没有，则创建默认的 view。

- **业务逻辑**

    不要重载该方法，创建 view 可以在viewDidLoad中

- **注意事项**

    不要创建view。不要调用 self.view。要调用都在 [super loadView]后。

## viewDidLoad

> 加载view完成。

- **业务逻辑**

    创建附加 view，但是 self.view 的 frame 不可用。

- **注意事项**

    该方法可能调用多次。
    
## viewWillApper

> view被添加到superview之前，切换动画之前调用。

- **业务逻辑**

    显示前的处理。如键盘弹出，状态条和navigationbar颜色。

- **注意事项**

    此时 View的 frame 不定，不能利用frame的值
    
## viewDidApper

> view 已经显示，动画切换完成

- **业务逻辑**

    业务处理 ，可以使用 self.view 的 frame

- **注意事项**

    暂无
    
## viewWillDisapper

> view移出之前，还未调用 removeFromSuperView

- **业务逻辑**

    根据具体业务处理

- **注意事项**

    暂无

## viewDidDisapper


> view移出完成，动画完成。

- **业务逻辑**

    处理view 不显示时的一些业务逻辑。

- **注意事项**

    暂无

## viewDidUnload


> 一般发生在内存警告时，view置为nil

- **业务逻辑**

    可以释放其他view，比如viewcontroller的 self.view上加了一个label，而且这个label是viewcontroller的属性，那么你要把这个属性设置成nil，以免占用不必要的内存，而这个label在viewDidLoad时会重新创建。

- **注意事项**

    暂无
    
## dealloc

> 销毁时调用

- **业务逻辑**

    移除观察者，定时器等

- **注意事项**

    在 ARC 环境下，不能主动调用 dealloc 方法。