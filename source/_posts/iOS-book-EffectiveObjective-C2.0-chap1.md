---
title: EffectiveObjective-C2.0 笔记 - 第一部分
date: 2018-06-16 14:25:22
categories: iOS
tags: [iOS,Effective Objective-C 2.0]
---

# EffectiveObjective-C2.0 笔记 - 第一部分
---

# 1.1-了解Objective-C

## 了解Objective-C 语言的起源

**1. Objective-C（以下简称Oc）是在C语言的基础上添加了面向对象特性。**

> Oc是C语言的超集（superset），因此C语言的所有功能特性都可以适用于Oc。

**2. Oc是使用“消息结构”（messaging structure)，而非常见的“函数调用”（function calling）。它们区别像这样**

```
// message structure 
Object *obj = [Object new];
[obj performWith:para1 and:para2]

// 其特性就是“运行时组件（Runtime）”，其本质上就是一种与开发者所编代码相链接的 “动态库”（dynamic libary），其代码能把开发者编写的所有程序粘合起来。
// 运行时所执行的代码由运行环境决定，动态特性明显，但是有些问题编译期间无法发现。
// 所有方法，都是运行时去查找，运行。接收消息的对象也要在运行时去查找。这时候就可能出问题。见后面。

// functions calling
Object *obj = new Object;
obj->perform(para1,para2);

// 与消息型相反，函数方法都有编译器编译的时候实现，可以预先发现一些潜在问题。
// 运行时所执行的代码由编译器决定；
// 如果是多态方法，运行时就会去“虚方法表（virtual table）”查找出具体哪一个函数；
```

**3. Oc中的对象总是分配在“堆空间”（heap space），不会分配到“栈”（stack）上。**

> Oc 将堆内存管理抽象出来了，不需要用malloc 及free 来分配或释放对象所占内存，Oc
运行期环境把这部分工作抽象成一套内存管理架构，叫 ”引用计数“ 。一个例子

```
NSString *someString = @"The string";
NSString *anotherString = someString;

```

![image](http://ot8psglzx.bkt.clouddn.com/QQ%E6%88%AA%E5%9B%BE20180914225911.png)

**4. 能用C的结构体，不用对象，结构体比对象更有效率。**

> 对象要分配空间，释放空间,而结构体不需要。结构体储存在栈空间。

**5. 非对象类型（nonobject type），分配在栈上，在其栈帧弹出时自动清理。**

# 1.2-头文件

**核心点：在类的头文件中尽量少引用其他头文件**

**1. Oc 中编写类方式与 C和C++一样，使用头文件（header file）、实现文件（implementation file）来区隔代码。**

**2. 引用其他类，使用@class xxx的向前声明方式（forward declaring）。**

> 也可以使用#import或#include，但是不够优雅，这里就要知道引用 类的具体细节，这里会引用到引用类的具体实现，会增加编译时间。使用@clss 还可以减少两个类之间的耦合。

**3.应该将引入头文件的时机尽量延后（放在实现文件），只有确有需要的时候才引用，这样子可以减少类的使用者所需引用的头文件数量。缩短编译时间。**

- 可以使用@class时，首选@class。

- 只有在迫不得已的时候才用#import (如：继承，实现协议)。

- 协议建议放在单独的一个头文件。避免引入协议时，引入头文件中等其他内容。

-  使用@class 可以减少.h中对其他类的依赖、减少链接到其他类所需要的时间，从而降低编译时间。

**4. 两个类互相引用时： A类中引用B类，B类中也引用A类。必须用@class，不然会出现循环引用。**

### 用#import 而不用 #include

- import可以避免重复引用

- 如果用#include的话，需要进行避免重复的宏定义

```
#ifndef HEADER
#define HEADER

xxx

#endif
```

# 1.3-字面量语法、常量、枚举

## 一、尽量用字面量语法，便于理解

**1. 字面数值**

```
// 字面量语法
NSNumber *itemNo = @1;

// 传统声明
NSNumber *itemNo = [NSNumber numberWithInt:1];
```

**2. 字面量数组**

不能有nil值，nil值为结尾标示

```
NSArray *arr1 = @[@"1",@"2"];

NSArray *arr2 = [NSArray arrayWithObjects:@"1",@"2",nil]; // 注意nil结尾
```

**3. 字面量字典**

不能有nil值，nil值为结尾标示

```
NSDictionary *dic = @{@"key1":@"val1",
                      @"key2":@"val2"};
```

**4. 可变数组和字典**

字面创建的都是不可变类型，如果想创建可变类型，需要mutableCopy

```
NSMutableDictionary *dic = [@{@"key1":@"val1",
                              @"key2":@"val2"} mutableCopy];
```

**5. 字符串字面量创建的是常量，对象不在持有了也不会立马被释放**

```
// Oc会做字符串的编译单元，而且会合并相同字符串的编译单元，来减少额外的消耗去链接这些编译单元。

NSString str1 = @“i am yun”;
NSString str2 = @“i am yun”;

// 此时，str1跟str2内存地址是一样的。

// 字符串常量创建后，不再修改。即使引用它的对象不再指向它，字符串常量也不会立即施放。
```

## 二、 多用类型常量，少用预处理

**1. 不要用预处理命令定义常量，用静态常量代替**

预处理命令定义的，不含有类型信息

**2. 类内使用的常量，定义在实现文件，可用k做前缀**

```
// in the implementation file
static const int kTimeItv = 1;
```

**3. 如果需要其他类引用常量，在接口用extern定义，在实现文件实现，可用类名做前缀，如通知键值**

```
// in the interface
extern const NSString *notiKey;

// in the implementation file
const  NSString *notiKey = @"notiKey";
```

**4. 编译器会在 “数据段”（data section）为字符串分配存储空间，这里在上面C 语言的内存模型有讲，数据段通常是指用来存放程序中已初始化的全局变量的一块内存区域。数据段属于静态内存分配。**

## 三、 用枚举表示状态、选项、状态码

**1. 用宏来定义枚举类型**

这些宏具备向后兼容（backward compatibility）能力，如果目标平台编译器支持新标准，那就使用新式语法，否则改用旧式语法。

- NS_ENUM宏 定义通用枚举

```
typedef NS_ENUM(NSInteger, NSWritingDirection) {
    NSWritingDirectionNatural     = -1,  //值为-1    
    NSWritingDirectionLeftToRight = 0,   //值为0
    NSWritingDirectionRightToLeft = 1    //值为1       
};

```

- NS_OPTIONS宏 定义位移枚举

```
typedef NS_OPTIONS(NSUInteger, UISwipeGestureRecognizerDirection) {
    UISwipeGestureRecognizerDirectionNone  = 0,       //值为0
    UISwipeGestureRecognizerDirectionRight = 1 << 0,  //值为2的0次方
    UISwipeGestureRecognizerDirectionLeft  = 1 << 1,  //值为2的1次方
    UISwipeGestureRecognizerDirectionUp    = 1 << 2,  //值为2的2次方
    UISwipeGestureRecognizerDirectionDown  = 1 << 3   //值为2的3次方
};
```

**2. 在switch 语句中，最好不要有default 分支，这样子要做到处理所有样式，这样子在新家类型的时候，没有default 编译器会发出警告，让我们注意到。**

**3. 实现枚举所用的数据类型取决于编译器，不过其二进制位（bit）的个数必须能完全表示下枚举编号才行，一个字节含8个二进制位，所以至多能表示256（2^8^）个枚举变量。**



































































