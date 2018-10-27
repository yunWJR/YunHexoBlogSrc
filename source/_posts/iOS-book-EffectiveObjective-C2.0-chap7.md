---
title: EffectiveObjective-C2.0 笔记 - 第七部分
date: 2018-10-12 21:43:22
categories: iOS
tags: [iOS,Effective Objective-C 2.0]
---

**EffectiveObjective-C2.0 笔记 - 第七部分**



# 7. 系统框架

## 7.1 熟悉系统框架

**1. 框架：将一系列代码封装成动态库，并在其中放入描述其接口的头文件。**

> 平时我们第三方框架用的是静态库，因为iOS 应用程序不允许其中包含动态库。

**2. Foundation、CoreFoundation 框架平时用的比较多，“无缝桥接” 可以将这两种框架的对象平滑转换。**

**3. 常用框架：**

   * CFNetwork
   * CoreAudio
   * AVFoundation
   * CoreData
   * CoreText


## 7.2 多用块枚举，少用for 循环

**遍历collection 有四种方式。最基本的办法就是for 循环，其次是NSEnumerator 遍历法及快速遍历法，最新、最先进的方式则是 “块枚举法”。**

**1. for 循环**

简单粗暴，遍历数组还可以，但是对于遍历字典或者set，就不太友好。  

**2. 使用Objective-C 1.0 的NSEnumerator 来遍历**

```objective-c
	NSArray *array = @[@"A",@"B",@"C"];
	NSEnumerator *enumerator = [array objectEnumerator];
	NSString *string;
	while ((string = [enumerator nextObject]) != nil) {
		NSLog(@"%@",string);
	}
```

这种遍历使用相对比较统一，数组、字典和set 都可以这样子写，并且还有多种 “枚举器” 可供使用，例如反向遍历数组的枚举器。

**3. 快速遍历**

```objective-c
    for (<#type *object#> in <#collection#>) {
        <#statements#>
    }
```

for in  这个更加简洁，如果某个类的对象支持快速遍历，那么就可以宣称自己遵从名为NSFastEnumeration 的协议，从而令开发者可以采用此语法来迭代改对象。此协议只定义了一个方法：

```objective-c
- (NSUInteger)countByEnumeratingWithState:(NSFastEnumerationState *)state objects:(id __unsafe_unretained _Nullable [_Nonnull])buffer count:(NSUInteger)len;
```

由于NSEnumerator 对象也实现了NSFastEnumeration 协议，所以能用来执行快速遍历。但是快速遍历拿不到当前操作对象的下标。

```objective-c
    NSArray *array = @[@"A",@"B",@"C"];
    for (NSString *string in [array reverseObjectEnumerator]) {
        NSLog(@"%@",string);
    }	
```

**4. 基于块的遍历方式**

```objective-c
    NSArray *array = @[@"A",@"B",@"C"];
    [array enumerateObjectsUsingBlock:^(id  _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
        
    }];

    [array enumerateObjectsWithOptions:NSEnumerationReverse usingBlock:^(id  _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
        
    }];
```

“块枚举法” 本身就能通过GCD来并发执行遍历操作，无须另行编写代码。而采用其他遍历则无法轻易实现这一点。

此方式对于其他相比，在遍历时候可以直接在块中获取更多信息，而且这种对于字典的遍历也是非常友好的，一次性可以返回键和值。并且还可以支持反向遍历。

## 7.3 对自定义其内存管理语义的collection使用无缝桥接

**无缝桥接**

使用 “无缝桥接” 计数，可以在定义于Foundation框架中的Objective-C类和定义于CoreFoundation框架中的C数据结构之间相互转换。

**1. 三种转换方式**

   * __bridge 只是声明类型转变，但是不做内存管理规则的转变
   * __bridge_retained 表示将指针类型转变的同时，将内存管理的责任由原来的Objective-C 交给Core Foundation 来处理，也就是ARC 转变成 MRC
   * __bridge_transfer 表示将管理的责任由Core Foundation 转交给Objective-C，即将MRC转变成ARC

2. Foundation中字典对象无缝桥接：

> Foundation中字典对象，对其键的内存管理语义为 “拷贝”，而值的语义是 “保留”。只能通过强大的无缝桥接技术，否则无法改变其语义。

   CoreFoundation 框架的字典类型是CFDictionary，可变版本是CFMutableDictionary。

   ```objective-c
   //CFMutableDictionary 用CFDictionaryCreateMutable 来创建
   //用CFDictionaryCreateMutable 定义
   CFMutableDictionaryRef CFDictionaryCreateMutable (
     CFAllocatorRef allocator, 
     CFIndex capacity, 
     const CFDictionaryKeyCallBacks *keyCallBacks, 
     const CFDictionaryValueCallBacks *valueCallBacks
   );

   /*CFAllocatorRef 表示将要使用的内存分配器，CoreFoundation 对象里的数据结构需要占用内存，而分配器负责分配及回收这些内存，一般传NULL，表示采用默认的分配器。

   CFIndex 表示字典的初始大小，跟我们Foundation 字典的创建一样，并不限制最大容量 就是预先分配内存

   最后两个参数都是指向结构体的指针，定义了很多回调函数，用于指示字典中的键和值遇到各种事件时应该执行何种操作。

   CFDictionaryKeyCallBacks 的结构体定义
   typedef struct {
       CFIndex				version;
       CFDictionaryRetainCallBack		retain;
       CFDictionaryReleaseCallBack		release;
       CFDictionaryCopyDescriptionCallBack	copyDescription;
       CFDictionaryEqualCallBack		equal;
       CFDictionaryHashCallBack		hash;
   } CFDictionaryKeyCallBacks;

   CFDictionaryValueCallBacks 的结构体定义
   typedef struct {
       CFIndex				version;
       CFDictionaryRetainCallBack		retain;
       CFDictionaryReleaseCallBack		release;
       CFDictionaryCopyDescriptionCallBack	copyDescription;
       CFDictionaryEqualCallBack		equal;
   } CFDictionaryValueCallBacks;

   version 参数目前应设置为0，表示版本号；
   其他参数都是函数指针，例如，字典加入了新的键与值，那么就会调用retain 函数，定义如下：
   typedef const void *(*CFDictionaryRetainCallBack)(
   	CFAllocatorRef allocator, 
   	const void *value
   );
   retain 是个函数指针，其所指向的函数接受两个参数，其类型分别是CFAllocatorRef、const void *。传给此函数的value 参数表示即将加入字典中的键或值。而返回的void * 则表示加到字典里的最终值。我们可以这样子实现：
   const void *CustomCallback（CFAllocatorRef allocator，const void *value）{
   	return value;
   }
   如果用它充当retain 回调函数来创建字典，那么该字典就不会 “保留” 键和值。然后再利用无缝桥接搭配起来，就可以创建特殊的NSDictionary 对象，跟我们普通的字典不一样。

   开发者可以直接在CoreFoundation 层创建字典，于是就能修改内存管理语义，对键执行 “保留” 而非 “拷贝” 操作了。
   ```


## 7.4 构建缓存时选用NSCache而非 NSDictionay

**1. 实现缓存时应选用NSCache而非NSDictionary 对象。**

> NSCache 是专门来处理缓存的，在系统资源将要耗尽时，它可以自动删减缓存。而且是线程安全的，此外，它与字典不同，并不会拷贝健。

**2. 可以给NSCache 对象设置上限**

> 可以给NSCache 对象设置上限，用以限制缓存中的对象总个数及总成本，而这些尺度则定义了缓存删减其中对象的时机。但是绝对不要把这些尺度当成可靠的 “硬限制”，它们仅对NSCache其指导作用。

**3. NSPurgeableData 与 NSCache 搭配使用**

>  NSPurgeableData类是NSMutableData的子类，而且实现了NSDiscardableContent协议。将NSPurgeableData 与 NSCache 搭配使用，可实现自动清除数据的功能，也就是说，当NSPurgeableData 对象所占内存为系统丢弃时，该对象也会从缓存中移除。

**4. 如果缓存使用得当，那么应用程序的响应速度就能提高。**

> 只有那种 “重新计算起来很费事的” 数据，才值得放入缓存，比如那些需要从网络获取或从磁盘读取的数据。

## 7.5 精简initalize与load的实现代码

**1. 在加载阶段，如果类实现了load 方法，那么系统就会调用它。分类里也可以定义此方法，类的load 方法要比分类中的先调用。与其他方法不同，load 方法不参与覆写机制。**

- 对于加入运行期系统中的每个类及分类，必定会调用load这个方法，而且仅调用一次。意思就是程序启动的时候需要加载load方法，这个时候运行期系统也是出于 “脆弱状态”，在执行子类的load方法之前，必定会先执行所有超类的load 方法。

- 如果load 代码还依赖了其他类，那类的load 也必然会先执行，我们无法判断每个类的载入顺序，所以load 方法使用其他类是不安全的。

- load 方法不遵从继承规则，如果某个类没实现load 方法，那么不管其各级超类是否实现此方法，系统都不会调用。

- load 方法要实现的精简点，因为应用程序在执行load 方法会阻塞。load 一般作为调试用，很少用来做初始化操作。

**2. load 与initialize 方法都应该实现得精简一些，这有助于保持应用程序的响应能力，也能减少引入 “依赖环” 的几率。**

- 想执行与类相关的初始化操作，可以使用 `+(void)initialize`  这个方法，它跟load 有以下几个区别：

   * 这个方法是在首次使用这个类的时候调用，类似 “惰性调用” ，只有用到这个类才会调用。


   * 运行期在执行该方法的时候，是出于正常状态的，此时是可以安全调用任意类的任意方法，而且运行期系统会确保initialize 方法一定在 “线程安全的环境” 中执行。其他线程都要先阻塞，等initialize 执行完。
   * initialize 方法跟其他方法一样，某个类没有实现它，而超类方法实现了，那么就会运行超类的实现代码。

- 也就是说initalize 与 load 的实现代码要精简些。

- 若某个全局状态无法在编译期初始化，则可以放在initalize 里来做。（例如Objectice-C 对象，创建实例之前必须先激活运行期系统）


## 7.6 别忘了NSTimer会保留其目标对象

**1. 计时器放在运行循环里，它才能正常触发任务。**

``` 
  + scheduledTimerWithTimeInterval:target:selector:userInfo:repeats:
```

**2. 计时器保留环**

   计时器会保留其目标对象，等到自身 “失效” 时再释放此对象，调用invalidate 方法可令计时器失效，另外，一次性的计时器在触发完任务之后也会失效。设置成重复执行模式的计时器，要注意 “保留环” 问题。

**3. 如何解决外界不调用invalidate方法也不产生 “保留环” 的问题**。

   可以用块来解决这个问题，其实就是将timer的target 对象不要指向持有timer的对象，这里用的方法是让timer 的taerget 指向自己。

   ```
   //定义
   + (NSTimer *)my_scheduledTimerWithTimeInterval:(NSTimeInterval)ti
                                            block:(void(^)())block
                                          repeats:(BOOL)yesOrNo;

   //实现
   + (NSTimer *)my_scheduledTimerWithTimeInterval:(NSTimeInterval)ti
                                            block:(void(^)())block
                                          repeats:(BOOL)yesOrNo {
       return [self scheduledTimerWithTimeInterval:ti
                                            target:self
                                          selector:@selector(my_blockInvoke:)
                                          userInfo:[block copy]
                                           repeats:yesOrNo];
   }

   - (void)my_blockInvoke:(NSTimer *)timer {
       void (^block) () = timer.userInfo;
       if (block) {
           block();
       }
   }
   ```
















































































































































