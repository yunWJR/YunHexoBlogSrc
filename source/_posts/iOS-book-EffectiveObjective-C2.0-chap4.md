---
title: EffectiveObjective-C2.0 笔记 - 第四部分
date: 2018-09-24 20:11:32
categories: iOS
tags: [iOS,Effective Objective-C 2.0]
---

**EffectiveObjective-C2.0 笔记 - 第四部分**


# 4 协议与分类

## 4.1 通过委托与数据源协议进行对象间通信

**1. 协议（protocol）类似 java 的接口（interface）。Objective-C 不支持多重继承，但我们可以把某个类应该实现的方法定义在一系列的协议里面。**

2. Objective-C 可以使用 “委托模式”（Delegate pattern）的编程设计模式来实现对象间的通信：

> 定义一套接口，某对象若想接受另一个对象的委托，则需遵从此接口，以便成为其 “委托对象”（delegate）。Objective-C 一般利用 “协议” 机制来实现此模式。

定义协议：

```
//采用 “驼峰法” 命名，最后加上 Delegate 一词
@protocol EOCNetworkingFetcherDelegate<NSObject>

// 可选实现的方法
@optional
- (void)newworkingFetcher:(EOCNetworkingFetcher *)fetcher
            didRecevieData:(NSData *)data;

// 必须实现的方法
@required
- (void)newworkingFetcher:(EOCNetworkingFetcher *)fetcher
         didFailWithError:(NSError *)error;

@end

@interface EOCNetworkingFetcher : NSObject

// weak，避免循环引用
@property (nonatomic,weak) id delegate;

@end
```

**3. 如果要在委托对象上调用可选方法，那么必须提前使用类型信息查询方法，判断这个委托对象能否响应相关的选择子。**

```
NSData *data;
if([_delegate respondsToSelector:@selector(networkFetcher:didRecevieData:)]){
    [_delegate networkFetcher:self didRecevieData:data];
}
```

**4. delegate 里的方法也可以用于从委托对象中获取信息（数据源模式）。**

**5. 在实现委托模式和数据源模式的时，协议中的方法是可选的，我们就会写出大量这种判断代码：**

```
if([_delegate respondsToSelector:@selector(networkFetcher:didRecevieData:)]){
    [_delegate networkFetcher:self didRecevieData:data];
}
```

每次调用方法都会判断一次，其实除了第一次检测的结构有用，后续的检测很有可能都是多余的，因为委托对象本身没变，不太可能会一下子不响应，一下子响应的，所以我们这里可以把这个委托对象能否响应某个协议方法记录下来，以优化程序效率。
将方法响应能力缓存起来的最佳途径是使用 “位段”（bitfield）数据类型。我们可以把结构体中某个字段所占用的二进制位个数设为特定的值。

```
// 协议
@protocol EOCNetworkingFetcherDelegate<NSObject>

@optional
- (void) didReceiveData:(NSData *)data;

@end

@interface EOCNetworkingFetcher ()
// 定义位段
struct {
    unsigned int didReceiveData : 1;
} _delegateFlags
@end

// 设置
-(void)setDelegate:(id<EOCNetworkingFetcherDelegate>)delegate{
	_delegate = delegate;
	_delegateFlags.didReceiveData = [_delegate respondsToSelector:@selector(didRecevieData:)]
}

//使用
if(_delageteFlags.didReceiveData){
	[_delegate didRecevieData:data];
}
```

## 4.2 将类的实现代码分散到便于管理的数个分类之中

**1. 使用分类机制把类的实现代码划分成易于管理的小块。**

**2. 将应该视为 “私有” 的方法归入为叫Private 的分类中，以隐藏实现细节。**

**3. 分类原则上不能定义属性，当可以通过【关联对象】实现，参见《2.5-关联对象存放自定义数据》**

## 4.3 总是为第三方类的分类名称加前缀

**1. 分类机制常用于向无源码的既有类中新增新功能，但是在使用的时候要十分小心，不然很容易产生Bug。因为这个机制时在运行期系统加载分类时，将其方法直接加到原类中，这里要注意方法重名的问题，不然会覆盖原类中的同名方法。**

**2. 一般用前缀来区分各个分类的名称与其中所定义的方法。**

**3. 不要轻易去利用分类来覆盖方法，这里需要慎重考虑。**

## 4.4 勿在分类中声明属性

**1. 可以利用运行期的关联对象机制，为分类声明属性，但是这种做法要尽量避免。**

> 因为除了 "class-continuation 分类" 之外，其他分类都无法向类中新增实例变量，因此，他们无法把实现属性所需的实例变量合成出来。

**2. 利用关联对象机制可以解决分类中不能合成实例变量的问题。**

> 自己实现存取方法，但是要注意该属性的内存管理语义（属性特质）。

```
@property (nonatomic,copy) NSString *name;
static const void *kViewControllerName = &kViewControllerName;
- (void)setName:(NSString *)name {
    objc_setAssociatedObject(self, kViewControllerName, name, OBJC_ASSOCIATION_COPY_NONATOMIC);
}
- (NSString *)name {
    NSString *myName = objc_getAssociatedObject(self, kViewControllerName);
    return myName;
}
```

**3. 在可以修改源代码的情况下，尽量把属性定义在主接口中。**

> 这里是唯一能够定义实例变量的地方，属性只是定义实例变量及相关存取方法所用的 “语法糖”。

## 4.5 使用 ”class-continuation 分类“ 隐藏实现细节


**“class-continuation 分类”必须定义在本身类的实现文件中，而且这里是唯一可以声明实例变量的分类(除关联对象机制)。**

> 而且此分类没有特定的实现文件，这个分类也没有名字。这里可以定义实例变量的原因是 “ 稳固的ABI” 机制，我们无须知道对象的大小就可以直接使用它。

```
@interface EOCPerson ()
@end
```

**私有的方法、协议、变量等，都可以定义在“class-continuation 分类”**


**1. 可以将不需要要暴露给外界知道的实例变量及方法写在 “class-continuation 分类” 中。**

**2.  可以利用 “class-continuation 分类” 把引用C++ 类的细节写到实现文件中，这样子别的类引用这个类就不会受到影响，甚至都不知道这个类底层实现混有C++ 代码。**

> 编写Objective-C++ 代码时候，使用 “class-continuation 分类” 会十分方便。因为对于引用了C++的文件的实现文件需要用.mm 为扩展名，表示编译器应该将此文件按照Objective-C++ 来编译。C++ 类必须完全引入，编译器要完整地解析其定义才能得知这个C++ 对象的实例变量大小。如果把对C++ 类的引用写在头文件的话，其他引用到这个类也会引用到这个C++ 类，就也需要编译成Objective-C++ 才行，这样子很容易失控。

**3. 使用 “class-continuation 分类” 还可以将头文件声明 “只读” 的属性扩展成 “可读写”，以便在类的内部可以设置其值。**

> 我们通常不直接访问实例变量，而是通过设置方法来做，因为这样子可以触发 “键值观测” （Key-Value Observing，KVO）通知。

**4. 若对象所遵循的协议只应视为私有，也可以同过“class-continuation 分类” 来隐藏。**

## 4.6 通过协议提供匿名对象

```
@property (nonatomic,weak) id delegate;
```

该属性类型是id的，所以实际上任何类的都能充当这一属性，即便该类不继承NSObject 也可以，只要遵循EOCDelegae 协议就可以了，对于具备此属性的类来说，delegate 就是 “匿名的”。

**1. 使用匿名对象来隐藏类型名称（或类名）。**

**2. 如果具体类型不重要，重要的是对象能够响应（定义在协议里的）特定方法，那么可使用匿名对象来表示。**


























































































































































































