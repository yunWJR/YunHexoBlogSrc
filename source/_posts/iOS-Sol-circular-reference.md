---
title: iOS中的循环引用
date: 2018-04-20 09:17:29
categories: iOS
tags: [iOS,Objective-C,总结]
---

# iOS中的循环引用

## 1. 概述

iOS内存中的分区有：堆、栈、静态区。其中，栈和静态区是操作系统自己管理回收，不会造成循环引用。在堆中的相互引用无法回收，有可能造成循环引用。

> 循环引用的实质：多个对象相互之间有强引用，不能施放让系统回收。

> 解决循环引用一般是将 strong 引用改为 weak 引用。

## 2. 循环引用场景分析及解决方法

### 1）父类与子类

> 如：在使用UITableView 的时候，将 UITableView 给 Cell 使用，cell 中的 strong 引用会造成循环引用。

```
// controller
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath {
    TestTableViewCell *cell =[tableView dequeueReusableCellWithIdentifier:@"UITableViewCellId" forIndexPath:indexPath];
    cell.tableView = tableView;
    return cell;
}

// cell
@interface TestTableViewCell : UITableViewCell
@property (nonatomic, strong) UITableView *tableView; // strong 造成循环引用
@end
```

> 解决：strong 改为 weak

```
// cell
@interface TestTableViewCell : UITableViewCell
@property (nonatomic, weak) UITableView *tableView; // strong 改为 weak
@end
```

### 2）block

> block在copy时都会对block内部用到的对象进行强引用的。

```
self.testObject.testCircleBlock = ^{
   [self doSomething];
};
```

self将block作为自己的属性变量，而在block的方法体里面又引用了 self 本身，此时就很简单的形成了一个循环引用。

应该将 self 改为弱引用

```
__weak typeof(self) weakSelf = self;
 self.testObject.testCircleBlock = ^{
      __strong typeof (weakSelf) strongSelf = weakSelf;
      [strongSelf doSomething];
};
```

> 在 ARC 中，在被拷贝的 block 中无论是直接引用 self 还是通过引用 self 的成员变量间接引用 self，该 block 都会 retain self。

- **快速定义宏**

```
    // weak obj
    /#define WEAK_OBJ(type)  __weak typeof(type) weak##type = type;

    // strong obj
    /#define STRONG_OBJ(type)  __strong typeof(type) str##type = weak##type;
```

### 3）Delegate

delegate 属性的声明如下：
```
@property (nonatomic, weak) id <TestDelegate> delegate;
```

如果将 weak 改为 strong，则会造成循环引用

```
// self -> AViewController
BViewController *bVc = [BViewController new];
bVc = self; 
[self.navigationController pushViewController: bVc animated:YES];

   // 假如是 strong 的情况
   // bVc.delegate ===> AViewController (也就是 A 的引用计数 + 1)
   // AViewController 本身又是引用了 <BViewControllerDelegate> ===> delegate 引用计数 + 1
   // 导致： AViewController <======> Delegate ，也就循环引用啦
```

### 4）NSTimer

NSTimer 的 target 对传入的参数都是强引用（即使是 weak 对象）

![](http://ot8psglzx.bkt.clouddn.com/784630-28d5d03d2d902860.png?imageMogr2/thumbnail/!70p)

解决办法: 《Effective Objective-C 》中的52条方法

```
#import <Foundation/Foundation.h>

@interface NSTimer (YPQBlocksSupport)

+ (NSTimer *)ypq_scheduledTimeWithTimeInterval:(NSTimeInterval)interval
                                         block:(void(^)())block
                                       repeats:(BOOL)repeats;

@end


#import "NSTimer+YPQBlocksSupport.h"

@implementation NSTimer (YPQBlocksSupport)


+ (NSTimer *)ypq_scheduledTimeWithTimeInterval:(NSTimeInterval)interval
                                         block:(void(^)())block
                                       repeats:(BOOL)repeats
{
    return [self scheduledTimerWithTimeInterval:interval
                                         target:self
                                       selector:@selector(ypq_blockInvoke:) userInfo:[block copy]
                                        repeats:repeats];
}

- (void)ypq_blockInvoke:(NSTimer *)timer
{
    void (^block)() = timer.userInfo;
    if(block)
    {
        block();
    }
}

@end
```

使用方式：

```
__weak ViewController * weakSelf = self;
[NSTimer ypq_scheduledTimeWithTimeInterval:4.0f
                                     block:^{
                                         ViewController * strongSelf = weakSelf;
                                         [strongSelf afterThreeSecondBeginAction];
                                     }
                                   repeats:YES];
```

> 计时器保留其目标对象，反复执行任务导致的循环，确实要注意，另外在dealloc的时候，不要忘了调用计时器中的 invalidate方法。