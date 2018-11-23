---
title: iOS 实现AOP编程(Objective-C)
date: 2018-11-23 17:26:56
categories: iOS
tags: [iOS,Objective-C]
---

# iOS 实现AOP编程(Objective-C)

## 一、AOP与OOP

* **OOP（Object Oriented Programming，面向对象编程）**

> OOP比较经典的程序设计思想，面向对象的特点是封装、多态和继承。面向对象设计时，每个对象职责不同，封装的功能也不同。这样就进行了解耦，增加了代码的重用性、灵活性和扩展性。

> 但这种方式也存在一个问题，比如，我们在两个类中，可能都需要在每个方法中进行日志记录（功能完全一样）。按OOP 方式，需要两个类的方法中都加入日志功能。这样就会有很多重复代码，当需要更改日志记录功能时，每个实现的类都需要更改。

> 一种解决方法：将日志功能写在一个独立的类中，然后再在这两个类中调用该类的日志记录功能。修改日志功能只需要修改单独的类即可。但是各个类与独立类有耦合，当有一个类需要增加或移除日志记录功能时，需要修改该类。另一种方法就是 AOP。

* **AOP（Aspect Oriented Program，面向切面编程）**

> AOP 思想是一种在不修改源代码的情况下给程序动态统一添加功能的一种技术。一般通过预编译方式和运行期动态代理实现程序功能的统一维护。

> 一般而言，我们管切入到指定类指定方法的代码片段称为切面，而切入到哪些类、哪些方法则叫切入点。

> AOP 与 OOP 配合，可以很好的分离应用的业务逻辑与系统级服务。有了AOP，我们就可以把几个类共有的代码，抽取到一个切片中，等到需要时再切入对象中去，从而改变其原有的行为。

## 二、 iOS实现 AOP

实现 AOP 需要语言支持对对象的动态扩展，正好 Objective-C的 Runtime 特性可以实现。现在有两种实现方式：

- **1. Method Swizzling**

* **2. 消息转发**

### 1. Method Swizzling 实现 AOP

在Objective-C中调用一个方法，其实是向一个对象发送消息，查找消息的唯一依据是selector的名字。

利用Objective-C的动态特性，可以实现在运行时偷换selector对应的方法实现。 

每个类都有一个方法列表，存放着selector的名字和方法实现的映射关系。IMP有点类似函数指针，指向具体的Method实现。 

* 每个类（Class）维护一张调度表（dispatch table）用于解析运行时发送的消息；
* 调度表中的每个实体（entry）都是一个方法（Method），其中key值是一个唯一的名字——选择器（SEL），它对应到一个实现（IMP - 实际上就是指向标准C函数的指针）。
 
Method Swizzling就是改变类中SEL 的具体实现函数IMP。


```
struct objc_method {
    SEL method_name             OBJC2_UNAVAILABLE; // selector 名字
    char *method_types          OBJC2_UNAVAILABLE;
    IMP method_imp              OBJC2_UNAVAILABLE; // IMP 实现方法，运行时可更改
}  

// 常用函数
- method_exchangeImplementations  // 交换2个方法中的IMP

- class_replaceMethod // 会调用class_addMethod和method_setImplementation，先实现方法，再设置IMP

- method_setImplementation // 直接设置某个方法的IMP
```

可参考[EffectiveObjective-C2.0 笔记 - 第二部分](https://yunwjr.github.io/2018/06/18/iOS-book-EffectiveObjective-C2.0-chap2/)



**示例 - 日志打印**

* **封装的 Swizzling 方法**

```
+ (void)swizzClass:(Class)classItem originSel:(SEL)originSel newSel:(SEL)newSel {
    Method orgMd = class_getInstanceMethod(classItem, originSel);
    Method newMd = class_getInstanceMethod(classItem, newSel);

    IMP newImp = method_getImplementation(newMd);

    // 检查源方法有没有实现
    // 如果是YES,表示originSel没有实现，则需要先实现，然后再设置Imp
    // 如果是NO,表示originSel已经有存在的实现方法，此时，只需要将orgMd和newMd互换就好
    BOOL isAddMdSuccess = class_addMethod(classItem, originSel, newImp, method_getTypeEncoding(newMd));

    if (isAddMdSuccess) {
        // 会调用class_addMethod和method_setImplementation，先实现方法，再设置IMP
        class_replaceMethod(classItem, originSel, newImp, method_getTypeEncoding(newMd));
    }
    else {
        // orgMd和newMd互换
        method_exchangeImplementations(orgMd, newMd);
    }
}
```

* **注意classItem，看你是替换类的方法，还是实例对象的放**

```
+ (Class)getClassItem {
    Class classItem = nil;

    //要特别注意你替换的方法到底是哪个性质的方法
    // When swizzling a Instance method, use the following:
    // 仅替换本实例方法，子类方法不变
    classItem = [self class];

    // When swizzling a class method, use the following:
    // 替换类方法
    classItem = object_getClass((id) self);

    return classItem;
}
```

* **在 load 中交换**

``` Objective-C
// load 中执行 Swizzling
+ (void)load {
    static dispatch_once_t onceToken;

    // dealloc是关键字，不能使用@selector(dealloc)
    SEL orgSel = NSSelectorFromString(@"dealloc");

    SEL newSel = @selector(swizzing_dealloc);

    // 保证仅执行一次
    dispatch_once(&onceToken, ^{
        [self swizzClass:[self class]
               originSel:orgSel
                  newSel:newSel];
    });
}

- (void)swizzing_dealloc {
    NSLog(@" ** %@ 释放了 %s", NSStringFromClass([self class]), __func__);

    // 交换后，就不能用 [self dealloc]
    [self swizzing_dealloc];
}
```

* **为什么在 load 中交换**

> +(void)load 方法只要类所在文件被引用就会被调用，在程序运行后立即执行（在main()之前执行），这样就可以在执行方法前，完成方法的替换。

> 另外 +(void)initialize 方是在类或者其子类的第一个方法被调用前调用，因为method swizzling会影响全局，+load能够保证在类初始化的时候就会被加载，这为改变系统行为提供了一些统一性。 但+initialize并不能保证在什么时候被调用——事实上也有可能永远也不会被调用，例如应用程序从未直接的给该类发送消息。


**使用注意点：**

1. Method Swizzling 需要在 + (void)load{}中使用

2. Method Swizzling 需要保证只执行一次。 需要使用 dispatch_once;

3. 注意Class的选择，类对象还是实例对象

4. Method Swizzling 是以替换 IMP 来实现动态修改代码，这样实现的 AOP 不优雅，使用消息转发可以更优雅。


### 2. 消息转发 实现 AOP

[Aspects](https://github.com/steipete/Aspects)是一个已经实现的 AOP 轮子。下面结合Aspects对消息转发的实现进行分析。

#### 2.1 实例方法的执行

Objective-C 中执行实例方式，其实是给对象发送一个消息（`id objc_msgSend ( id self, SEL cmd, ... )`），执行流程如下：

![](http://qnyunyun.yunsoho.cn/msg_send2.png)

* 对象实例(instance)收到消息（selector 选择子+参数）
* 根据对象实例的ISA找到类对象，在类对象中找与选择子名称相符的方法，如果找到，就调至执行代码
* 如果找不到，则根据类对象中的super_class指针找到父类的Class对象。一直找到NSObject的类对象
* 如果NSObject也无法找到这个选择子，则进入消息转发机制（message forwarding）
* 如果消息转发机制无法处理，则抛出异常: `doesNotRecognizeSelector:`

#### 2.2 消息转发机制

在Objective C的方法调用过程中，当无法响应一个selector时，在抛出异常之前会先进入消息转发机制。这里来详细讲解消息转发的过程：

关于消息转发，官方文档在这里： [Message Forwarding](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtForwarding.html#//apple_ref/doc/uid/TP40008048-CH105-SW2)

其他参考[二、 消息转发](https://yunwjr.github.io/2018/06/18/iOS-book-EffectiveObjective-C2.0-chap2/)

![](http://qnyunyun.yunsoho.cn/message%20forwarding.png)

在触发消息转发机制即`forwardInvocation:`之前，Runtime提供了两步来进行轻量级的动态处理这个selector.

* **1. 动态方法 `resolveInstanceMethod:`**

> Dynamically provides an implementation for a given selector for an instance method.

这个方法提供了一个机会：为当前类无法识别的SEL动态增加IMP。

比如：可以通过`class_addMethod `增加 IMP

```
void dynamicMethodIMP(id self, SEL _cmd) {/*...implementation...*/}

+ (BOOL)resolveInstanceMethod:(SEL)aSEL {
    if (aSEL == @selector(resolveThisMethodDynamically)) {
        class_addMethod([self class], aSEL, (IMP) dynamicMethodIMP, "v@:");
        return YES;
    }
    return [super resolveInstanceMethod:aSel];
}


// "v@:"表示方法参数编码，v表示Void，@表示OC对象，:表示SEL类型。

```

如果`resolveInstanceMethod `返回NO，则表示无法在这一步动态的添加方法，则进入下一步：

* **2. 备援接收者 `forwardingTargetForSelector:`**

> Returns the object to which unrecognized messages should first be directed.

这个方法提供了一个机会：把这个SEL转给其他接收者来处理。

比如

```
- (id)forwardingTargetForSelector:(SEL)aSelector {
    if (aSelector == @selector(dynamicSelector) &&
        [self.myObj respondsToSelector:@selector(dynamicSelector)]) {
        return self.myObj;
    }
    else {
        return [super forwardingTargetForSelector:aSelector];
    }
}
```

* **3. 消息转发 message forwarding**

如果上述两步都无法完成这个SEL的处理，则进入消息转发机制，消息转发机制有两个比较重要的方法：

* forwardInvocation: 具体的NSInvocaion
* methodSignatureForSelector: 返回SEL的方法签名

这里不得不提一下两个类：

* NSMethodSignature 用来表示方法的参数签名信息：返回值，参数数量和类型
* NSInvocaion SEL + 执行SEL的Target + 参数值

通常，拿到NSInvocaion对象后，我们可选择的进行如下操作

* 修改执行的SEL
* 修改执行的Target
* 修改传入的参数

然后调用：`[invocation invoke]`，来执行这个消息。

**`_objc_msgForward`**

我们知道，正常情况下SEL背后会对一个IMP，在OC中有一个特殊的IMP就是：`_objc_msgForward`。当执行`_objc_msgForward`时，会直接触发消息转发机制，即`forwardInvocation:`。


#### 2.3 Method Swizzling

上一节已经介绍了Method Swizzling，可以替换SEL 对应的IMP。

#### 2.4 Aspect 实现

使用Aspect，可以在一个OC方法执行前/后插入代码，也可以替换这个OC方法的实现。通过作者暴露的2个接口可以实现对实例和类的 Hook:

```
+ (id<AspectToken>)aspect_hookSelector:(SEL)selector
                           withOptions:(AspectOptions)options
                            usingBlock:(id)block
                                 error:(NSError **)error;
 
- (id<AspectToken>)aspect_hookSelector:(SEL)selector
                           withOptions:(AspectOptions)options
                            usingBlock:(id)block
                                 error:(NSError **)error;
```

下面以在ViewControler的viewWillAppear:方法之后插入一段代码为例，来讲解hook前后的变化

**1) 在没有hook之前，ViewController的SEL与IMP关系如下**

![](http://qnyunyun.yunsoho.cn/iOSAspect1.jpg?imageMogr2/thumbnail/!100p)

**2) 调用以下aspect来Hook viewWillAppear:后**

```
[ViewController aspect_hookSelector:@selector(viewWillAppear:)
                            withOptions:AspectPositionAfter
                             usingBlock:^{
                                 NSLog(@"Insert some code after ViewWillAppear");
                             } error:&error];

```

![](http://qnyunyun.yunsoho.cn/iOSAspect3.jpg?imageMogr2/thumbnail/!100p)

* 最初的viewWillAppear: 指向了_objc_msgForward
* 增加了aspects_viewWillAppear:,指向最初的viewWillAppear:的IMP
* 最初的forwardInvocation:指向了Aspect提供的一个C方法__ASPECTS_ARE_BEING_CALLED__
* 动态增加了aspects_forwardInvocation:,指向最初的forwardInvocation:的IMP

**3) hook后，一个viewWillAppear:的实际调用顺序：**

* object收到selector(viewWillAppear:)的消息
* 找到对应的IMP：_objc_msgForward，执行后触发消息转发机制。
* object收到forwardInvocation:消息
* 找到对应的IMP：__ASPECTS_ARE_BEING_CALLED__，执行IMP 
	* 向object对象发送aspects_viewWillAppear:,执行最初的viewWillAppear方法的IMP
	* 执行插入的block代码
	* 如果ViewController无法响应aspects_viewWillAppear，则向object对象发送__aspects_forwardInvocation:来执行最初的forwardInvocation IMP

> 所以，Aspects是采用了集中式的hook方式，所有的调用最后走的都是一个C函数__ASPECTS_ARE_BEING_CALLED__。

##### 2.4.1 核心类/数据结构

![](http://qnyunyun.yunsoho.cn/iOSAspect5.jpg?imageMogr2/thumbnail/!100p)

**1） Aspects 内部定义了两个协议：**

* **AspectToken** 

AspectToken 协议旨在让使用者可以灵活的注销之前添加过的 Hook

```
/// 用于注销 Hook
@protocol AspectToken /// 注销一个 aspect.
/// 返回 YES 表示注销成功，否则返回 NO
- (BOOL)remove;
@end
```

* **AspectInfo** 

AspectInfo 协议旨在规范对一个切面，即 aspect 的 Hook 内部信息的纰漏，在 Hook 时添加切面的 Block 第一个参数就遵守此协议。

```
/// AspectInfo 协议是嵌入 Hook 的Block的第一个参数。
@protocol AspectInfo /// 当前被 Hook 的实例
- (id)instance;
/// 被 Hook 方法的原始 invocation
- (NSInvocation *)originalInvocation;
/// 所有方法参数（装箱之后的）惰性执行
- (NSArray *)arguments;
@end
```

**2） Aspects 内部还定义了 4 个类：**

* **AspectInfo** 

切面信息：NSInvocation的容器，表示一个执行的Command，遵循 AspectInfo 协议。AspectInfo 扮演了一个提供 Hook 信息的角色。

```
@interface AspectInfo : NSObject <AspectInfo>
@property (nonatomic, unsafe_unretained, readonly) id instance;
@property (nonatomic, strong, readonly) NSArray *arguments;
@property (nonatomic, strong, readonly) NSInvocation *originalInvocation;
@end

```

* **AspectIdentifier** 

切面 ID：代表一个Aspect的具体信息，包括被Hook的对象，SEL，插入的block等具体信息，遵循 AspectToken 协议。

```
@interface AspectIdentifier : NSObject
@property (nonatomic, assign) SEL selector;
@property (nonatomic, strong) id block;
@property (nonatomic, strong) NSMethodSignature *blockSignature;
@property (nonatomic, weak) id object;
@property (nonatomic, assign) AspectOptions options;
@end
```

* **AspectContainer** 

AspectIdentifier的容器：以SEL合成key，然后作为关联对象存储到对应的类/对象里。包括beforeAspects，insteadAspects，afterAspects

```
@interface AspectsContainer : NSObject
@property (atomic, copy) NSArray *beforeAspects;
@property (atomic, copy) NSArray *insteadAspects;
@property (atomic, copy) NSArray *afterAspects;
@end

```

* **AspectTracker** 

切面跟踪器：跟踪一个类的继承链中的hook状态：包括被hook的类，哪些SEL被hook了。

```
@interface AspectTracker : NSObject
@property (nonatomic, strong) Class trackedClass;
@property (nonatomic, readonly) NSString *trackedClassName;
@property (nonatomic, strong) NSMutableSet *selectorNames;
@property (nonatomic, strong) NSMutableDictionary *selectorNamesToSubclassTrackers;
@end

```

其原理大致为

```
// Add the selector as being modified.
currentClass = klass;
AspectTracker *parentTracker = nil;
do {
    AspectTracker *tracker = swizzledClassesDict[currentClass];
    if (!tracker) {
        tracker = [[AspectTracker alloc] initWithTrackedClass:currentClass parent:parentTracker];
        swizzledClassesDict[(id)currentClass] = tracker;
    }
    [tracker.selectorNames addObject:selectorName];
    // All superclasses get marked as having a subclass that is modified.
    parentTracker = tracker;
}while ((currentClass = class_getSuperclass(currentClass)));
```

> AspectTracker 是从下而上追踪，最底层的 parentEntry 为 nil，父类的 parentEntry 为子类的 tracker。

**3）一个结构体：**

* AspectBlockRef - 即 _AspectBlock，充当内部 Block


**4）两个内部静态全局变量：**

* static NSMutableDictionary *swizzledClassesDict;
* static NSMutableSet *swizzledClasses;


##### 2.4.2 hook过程

**1. 对Class和MetaClass进行进行合法性检查，判断能否hook，规则如下**

* retain,release,autorelease,forwoardInvocation:不能被hook
* dealloc只能在方法前hook
* 类的继承关系中，同一个方法只能被hook一次

**2. 创建AspectsContainer对象，以aspects_ + SEL为key，作为关联对象依附到被hook 的对象上**

```
objc_setAssociatedObject(self, aliasSelector, aspectContainer, OBJC_ASSOCIATION_RETAIN);
```

**3. 创建AspectIdentifier对象，并且添加到AspectsContainer对象里存储起来。这个过程分为两步** 

* 生成block的方法签名NSMethodSignature
* 对比block的方法签名和待hook的方法签名是否兼容（参数个数，按照顺序的类型）

**4. 根据hook实例对象/类对象／类元对象的方法做不同处理。其中，对于上文以类方法来hook的时候，分为两步**

* hook类对象的forwoardInvocation:方法，指向一个静态的C方法，并且创建一个aspects_ forwoardInvocation:动态添加到之前的类中

```
IMP originalImplementation = class_replaceMethod(klass, @selector(forwardInvocation:), (IMP)__ASPECTS_ARE_BEING_CALLED__, "v@:@");
if (originalImplementation) {
    class_addMethod(klass, NSSelectorFromString(AspectsForwardInvocationSelectorName), originalImplementation, "v@:@");
}
```

* hook类对象的viewWillAppear:方法让其指向_objc_msgForward,动态添加aspects_viewWillAppear:指向最初的viewWillAppear:实现

##### 2.4.3 Hook实例的方法

> Aspects支持只hook一个对象的实例方法

只不过在第4步略有出入，当hook一个对象的实例方法的时候：

* 新建一个子类，_Aspects_ViewController,并且按照上述的方式hook forwoardInvocation:
* hook _Aspects_ViewController的class方法，让其返回ViewController
* hook 子类的类元对象，让其返回ViewController
* 调用objc_setClass来修改ViewController的类为_Aspects_ViewController

> 这样做，就可以通过object_getClass(self)获得类名，然后看看是否有前缀类名来判断是否被hook过了


##### 2.4.4 其他

**1) object_getClass/与self.class的区别**

* object_getClass获得的是isa的指向
* self.class则不一样，当self是实例对象的时候，返回的是类对象，否则则返回自身。

比如：

```
TestClass * testObj = [[TestClass alloc] init];
//Same
logAddress([testObj class]);
logAddress([TestClass class]);

//Not same
logAddress(object_getClass(testObj));
logAddress(object_getClass([TestClass class]));
```

输出

```
2017-05-22 22:41:48.216 OCTest[899:25934] 0x107d10930
2017-05-22 22:41:48.216 OCTest[899:25934] 0x107d10930
2017-05-22 22:41:48.216 OCTest[899:25934] 0x107d10930
2017-05-22 22:41:49.061 OCTest[899:25934] 0x107d10908

```

**2) Block签名**

block因为背后其实是一个C结构体，结构体中存储着着一个函数指针来指向实际的方法体

Block的内存布局如下

```
typedef NS_OPTIONS(int, AspectBlockFlags) {
    AspectBlockFlagsHasCopyDisposeHelpers = (1 << 25),
    AspectBlockFlagsHasSignature          = (1 << 30)
};
typedef struct _AspectBlock {
    __unused Class isa;
    AspectBlockFlags flags;
    __unused int reserved;
    void (__unused *invoke)(struct _AspectBlock *block, ...);
    struct {
        unsigned long int reserved;
        unsigned long int size;
        // requires AspectBlockFlagsHasCopyDisposeHelpers
        void (*copy)(void *dst, const void *src);
        void (*dispose)(const void *);
        // requires AspectBlockFlagsHasSignature
        const char *signature;
        const char *layout;
    } *descriptor;
    // imported variables
} *AspectBlockRef;

```

对应生成NSMethodSignature的方法：

```
static NSMethodSignature *aspect_blockMethodSignature(id block, NSError **error) {
    AspectBlockRef layout = (__bridge void *)block;
    if (!(layout->flags & AspectBlockFlagsHasSignature)) {
        NSString *description = [NSString stringWithFormat:@"The block %@ doesn't contain a type signature.", block];
        AspectError(AspectErrorMissingBlockSignature, description);
        return nil;
    }
    void *desc = layout->descriptor;
    desc += 2 * sizeof(unsigned long int);
    if (layout->flags & AspectBlockFlagsHasCopyDisposeHelpers) {
        desc += 2 * sizeof(void *);
    }
    if (!desc) {
        NSString *description = [NSString stringWithFormat:@"The block %@ doesn't has a type signature.", block];
        AspectError(AspectErrorMissingBlockSignature, description);
        return nil;
    }
    const char *signature = (*(const char **)desc);
    return [NSMethodSignature signatureWithObjCTypes:signature];
}

```

**3) 效率**

> 消息转发机制相对于正常的方法调用来说是比较昂贵的，所以一定不要用消息转发机制来处理那些一秒钟成百上千次的调用。

