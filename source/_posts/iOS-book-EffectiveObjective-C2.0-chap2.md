---
title: EffectiveObjective-C2.0 笔记 - 第二部分
date: 2018-06-18 11:22:23
categories: iOS
tags: [iOS,Effective Objective-C 2.0]
---

# EffectiveObjective-C2.0 笔记 - 第二部分

# 2.1 属性

1. "对象"（object）就是 "基本构造单元"（building block），开发者可以通过对象来存储并传递数据。

> 在对象直接传递数据并执行任务的过程就叫做 "消息传递"（Messaging）。

2. 程序运行起来后，为其提供相关支持的代码叫做"运行期环境"(runtime)，它提供一些使得对象之间能够传递消息的重要函数。

> 理解运行期环境，可以帮你写出高效且易维护的代码。 

3. Oc编译采用“应用程序二进制接口”（Application Binary Interface，ABI）

> 把实例变量当作一种存储偏移量所用的 “特殊变量”（speacial variable），交由 “类对象”（class object）保管。偏移量会在运行期查找，这样子总能找到正确的偏移量，这是稳固。

> 如果对象布局在编译器就固定了，访问变量时，编译器会使用 “偏移量”（offset）来计算，这个偏移量是 “硬编码”（hardcode），表示该变量距离存放对象的内存区域的起始地址有多远。 存在一个问题：如果代码使用了编译期计算出来的偏移量，那么修改类定义之后必须重新编译，否则就会出错。

### @property

1. 用于声明属性，自动添加实例变量，以下划线开头，自动实现属性的读写方法。

2. 在实现文件中可以通过@synthesize 语法来指定实例变量的名字

```
@implementation EOCPerson
@synthesize name = _myName;
@end
```

3. @dynamic 关键字会告诉编译器：不要自动创建实现属性所用的实例变量，也不要为其创建存取方法

## 属性特质

原子性、读写权限、内存管理语义、方法名、其他。

### 1、原子性

- atomic -默认

    > 原子性，会生成读写锁，读写安全（线程不一定安全），占用资源、效率一般。

- nonatomic 

    > 非原子、效率高、读写不安全

### 2、读写权限

- readwrite -默认

    > 读写

- readonly 

    > 只读

### 3、内存管理

MRC时，有assign、retain、copy，ARC加入了strong、weak

- assign -值类型默认

    > 简单赋值、用于值类型，如CGFloat、NSInteger等

- strong (同retain -MRC) -引用类型默认

    > 强引用、用于引用类型
    
    > 赋值时，保留新值，新值引用计数+1，释放旧值（引用计数-1）。
    
    > 用于所有的实例变量和局部变量、其他常规对象引用。
    
    > 注意：可变对象应该使用strong，如NSMultiString，NSMultiArray

- copy 

    > 复制、用于引用类型

    > 赋值时，拷贝新值（新对象引用计数为1），释放旧值（引用计数-1），不改变新值（引用计数不变）。
    
    > copy的本质为复制该内存所存储的内容，重新创建一个对象赋给其相同的内容，对于实现了NSCopying协议的对象有效。
    
    > 用于不可变对象：NSString、block、NSArray、NSDictionary等
    
    > 注意：用于可变对象时，设置值后，变为不可变对象
    
- weak 

    > 弱引用、用于引用类型
    
    > 赋值时、单纯的引用新对象地址，不改变新对象（引用计数不变），不改变旧对象（引用计数不变）

    > 当引用对象释放后，其值置为nil

- __unsafe_unretained 

    > 类似assign、适用于引用类型、不安全的弱引用
    
    > 功能类似于weak、对象摧毁后，不置nil、不安全，可用weak代替

### 4、方法名

- getter=\<name>

    > 
    ```
    @property (nonatomic, getter=isOn) BOOL on;
    ```

- setter=\<name>

    > 
    ```
    @property (nonatomic, setter=setOnState) BOOL on;
    ```

### 5、其他

nonnull, null_resettable, nullable



# 2.2 对象访问

1. 对象访问有两种、一种是实例访问、一种是属性的读写方法访问。

* 一般可以这样做：读取时，通过实例读取、写入时，通过属性方法写入。初始化时，都用实例。

* 一般外部访问时，通过属性访问。

* 内部访问时、无特殊情况，通过实例访问。

2. 具体情况应该根据他们的特点来定：

* 实例访问不通过属性方法派发、效率高。

* 实例访问、不触发“键值观测”，无法满足某些场景。

* 初始化方法中，尽量实例访问，避免子类重写设置方法，导致出错。

* 如果待初始化的实例声明在超类中，而我们又无法在子类直接访问此实例变量的话，那么就需要调用 “设置方法” 了。

* 在 “惰性初始化”（lazy initialization），必须通过 “获取方法” 来访问属性，不然实例变量永远不会初始化。
```
-(EOCBrain *)brain{
    if(!_brain){
        _brain = [EOCBrain new];
    }
    
    return _brain;
}
```

# 2.3 对象同等性

1. == 与 isEqual

- ==

    > == 用于值对象时，可以直接判断值是否相等。

    > == 用于引用对象时，是判断两个对象的指针是否相等（为同一个对象），不能判断其内容等同。

- isEqual 

    > 用于引用类型、判断内容是否等同、常需要ovewrite该方法。

2. NSObject 协议中有两个用于判断等同性的关键方法：
```
- (BOOL)isEqual:(id)object;
- (NSUInteger)hash;
```
> 如果 “isEqual” 方法判定两个对象相等，那么其hash 方法也必须返回同一个值。

> 但是，如果两个对象的hash 方法返回同一个值，那么 “isEqual” 方法未必会认为两者相等。

hash方法实现的一些情况：
```
// 1-固定值
- (NSUInteger)hash {
    return 12312312;
}
// 这种会对collection使用这个对象产生性能问题。因为在collection 在检索哈希表的时，会用对象的哈希码来做索引，在set 集合中，会根据哈希码把对象分装到不同的数组里面，在添加新对象的时候，要根据其哈希码找对与之对应的数组，依次检查其中各个元素，看数组已有的对象是否和将要添加的新对象相等，如果相等，就说明添加的对象已经在set 集合中了，是添加失败的。（如果所有对象的hash 值对一样，这样子set 集合只会有一个数组，所有数据都在一起了，每次插入数据都会遍历这个数组，这样子就会出现性能问题）

// 2-组合值
- (NSUInteger)hash {
    NSString *stringToHash = [NSString stringWithFormat@"%@:%@",_firstName,_lastNmae];
    return [stringToHash hash];
}
//这样子能在一定情况下保证返回不同的哈希码，但是这里会存在创建字符串的开销，会比返回单一值要慢

// 3-位运算
- (NSUInteger)hash {
    return [self.firstName hash] ^ [self.lastNmae hash];
}

// ^为逐位逻辑运算符，它表示逐位非或（如果只有一个位为1，那么结果为1；否则为0。）。
//这样子可以保存较高的效率，又不会过于频繁的重复
```

3. 特定类所具有的等同性判定方法

```
isEqualToString、isEqualToArray、isEqualToDictionary
```

4. 容器中可变类的等同性

> 如果要把某个对象放入colloection ，其 hash 方法的生成策略就应该保证在放入colloection 后，hash 值不再改变。不然会出现问题。


# 2.4 类族模式

1. “类族” （class cluster）是一种很有用的模式（pattern），可以隐藏 “抽象基类” （abstract base class）背后的实现细节。

2. 用户无须自己创建子类实例，只需要调用基类方法来创建即可。

3. 如何创建类族

> 每个 “实体子类” 都从基类继承而来，“工厂模式” 是创建类族的办法之一，调用基类方法返回子类实例。

> 如果对象所属的类位于某个类族中，那么查询其类型信息要注意，你可能觉得自己创建了某个类的实例，然后实际上创建的却是其子类的实例。

```
-(BOOL) isKindOfClass: classObj; //判断是否是这个类或者这个类的子类的实例
-(BOOL) isMemberOfClass: classObj; //判断是否是这个类的实例
```

# 关联对象存放自定义数据

1. 关联对象 

可以给某个对象关联许多其他对象，这些对象通过“键”来区分。存储对象值的时候，可以指明“存储策略”，用以维护相应的“内存管理语义”。

```
OBJC_ASSOCIATION_ASSIGN --- assign
OBJC_ASSOCIATION_RETAIN_NONATOMIC --- nonatomic, retain
OBJC_ASSOCIATION_COPY_NONATOMIC --- nonatomic, copy
OBJC_ASSOCIATION_RETAIN --- retain
OBJC_ASSOCIATION_COPY --- copy
```

下列方法可以管理关联对象：

```
void objc_setAssociatedObject (id object, void *key, id value, objc_AssociationPolicy policy)
// 此方法以给定的键和策略为某对象设置关联对象值

id objc_getAssociatedObject(id object, void *key) 
// 此方法根据给定的键从某个对象中获取相应的关联对象值

void objc_removeAssociatedObject(id object) 
// 此方法移除指定对象的全部关联对象
```

若想令两个健匹配到相同的一个值，则二者必须是完全相同的指针才行。所以，在设置关联对象值时：**通常使用静态全局变量做键**。

# 2.6 消息

##  一. 消息传递（objc_msgSend）

1. 调用对象方法，在Objective-C 中叫做 “传递消息”（pass a message），消息有 “名称”（name）或“选择子”（selector），可以接受参数，而且可能还有返回值。

	objc_megSend 的原型：

```
// 方法原型
// messageName 叫做 selector（选择子），选择子和参数合起来称为"消息"。
id returnValue = [receiveObject messageName:parameter];

// 所有方法都是普通的 C 语言函数，方法转为标准的 C 语言函数如下：
// 是一个 “参数个数可变的函数”，能够接受两个或两个以上的参数,
// 第一个参数代表接收者，第二个参数代表选择子，后续参数就是参数。
void objc_msgSend(id self,SEL cmd,...)
```

2. objc_megSend 函数会依据接收者和选择子来调用适当的方法：

* 在接收者所属的类搜寻其 “方法列表”
* 找不到的话，就沿着继承体系继续向上查找
* 最终还是找不到相符的方法就执行 “消息转发”

3. 每个类里都有一张函数表，选择子的名称则是表的 “键”，对应的值都是指向函数的指针。objc_msgSend 等函数就是通过这个函数表来寻找应该执行的方法并执行跳转的。

4. objc_msgSend 会将匹配结果缓存在 “快速映射表”（fast map）里面，每个类都有这样子的一块缓存，接下来还向该类发送一样的消息，那么执行起来就很快了。

5. 这里有些特殊情况，需要由Objective-C 运行环境的另外一些函数来处理：

* objc_msgSend_stret ：如果待发送的消息要返回结构体，那么可以交由此函数处理。只有当CPU 寄存器能够容纳得下消息返回类型时，这个函数才能处理此消息。若是返回值无法容纳于CPU 寄存器（比如说返回的结构体太大了），那么就由另外一个函数执行派发。此时，那个函数会通过分配在栈上的某个变量来处理消息所返回的结构体。
* objc_msgSend_fpret：如果消息返回的是浮点数，可以交由此函数处理。这个函数是为了处理x86 等架构CPU 中某些令人惊讶的奇怪状况。
* objc_msgSendSuper：如果要给超类发消息，那么就交由此函数处理。

6. 如果某函数的最后一项操作是调用另外一个函数，那么就可以运用 “尾调用优化” 技术。编译器会生成跳转至另外一个函数所需的指令码，而且不会向调用栈推入新的 “栈帧”。


## 二、 消息转发

当对象接收到无法解读的消息后，就会启动 “消息转发”（message forwarding）机制，程序员可经由此过程告诉对象应该如何处理未知消息。

1. 消息转发分为两大阶段：

* 动态方法解析

> 第一阶段选征询接收者，所属的类，看其是否能动态添加方法，以处理当前这个 “未知的选择子“（unknown seletor），这叫做 ”动态方法解析“（dynamic method resolution）。

```
//表示这个类是否能新增一个方法来处理此选择子
+ (BOOL)resolveClassMethod:(SEL)sel
+ (BOOL)resolveInstanceMethod:(SEL)sel
```

* 第二阶段涉及 ”完整的消息转发机制“（full forwarding mechanism）。

> 如果运行期系统已经把第一阶段执行完了，那么接收者自己就无法再以动态新增方法的手段来响应包含该选择子的消息了。这里的第二阶段又分为下面两小步：

* 1) 备援的接收者

> 首先，请接收者看看有没其他对象能处理这条消息；若有，则运行期系统会把消息转给那个对象，于是消息转发过程结束，一切正常。

```
- (id)forwardingTargetForSelector:(SEL)aSelector
```

 * 2) 若没有 ”备援的接收者“（replacement receiver），则启动完整的消息转发机制。

 > 运行期系统会把与消息有关的全部细节都封装到NSInvocation 对象中，再给接受者最后一次机会，令其设法解决当前还未处理的这条消息。

```
- (void)forwardInvocation:(NSInvocation *)anInvocation
```











































































































