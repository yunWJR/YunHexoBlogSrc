---
title: EffectiveObjective-C2.0 笔记 - 第三部分
date: 2018-9-20 14:12:31
categories: iOS
tags: [iOS,Effective Objective-C 2.0]
---

**EffectiveObjective-C2.0 笔记 - 第三部分**


# 3 接口与 API 设计

## 3.1 用前缀避免命名空间冲突

**1.如果发生命名冲突（naming clash），那么应用程序的链接过程就会出错，因为出现了重复符号(duplicate symbol)。**

**2.应该为所有名称都加上适当的前缀，最好是**++三个字母以上++**做前缀，因为Apple宣称其保留使用所有 “两字母前缀”。**

**3.在类的实现文件所有的纯C 函数及全局变量，也是容易命名冲突的，在编译好的目标文件中，这些要算做 “顶级符号”（top-level symbol）。**

## 3.2 提供 “全能初始化方法”

**1.“全能初始化方法”（designated initializer）：为对象提供必要信息以便其能完成工作的初始化方法。**

**2.每个子类的全能初始化方法都应该调用其超类的对应方法，并逐层向上。**

- 在类中提供一个全能初始化方法，并于文档里指明。其他初始化方法均应调用此方法。
- 若全能初始化方法与超类不同，则需覆写超类中的对应方法。
- 如果超类的初始化方法不适用于子类，那么应该覆写这个超类方法，并在其中抛出异常。



## 3.3 实现 description 方法

**1.在调用NSLog(@"object = %@",onbject); 其实是调用了对象的description 方法。**

**2.在我们自定义类中，这样子打印输出信息有可能是这种object =，这个我们需要重写description 方法，让它返回我们需要的一些信息。**

**3.description 定义在NSObject 协议里面，因为NSObject 不是唯一的 “根类”，用继承不能很好的让其他类有这个方法**

> 例如：NSProxy 也是遵从了NSObject 协议的 “根类”。

**4.debugDescription 方法是开发者在调试器中以控制台命令打印对象时才调用的，默认是直接调用description 方法。**

**5.小技巧：可以在description 中用NSDictionary 的description 方法来输出，就是将信息用字典的形式来展示，这样子更加直观，也更加容易扩展。**

## 3.4 尽量使用不可变对象

**1.设计类的时候，用属性来封装数据，在用属性的时候，可将其声明为 “只读” ，避免外部不必要的修改**

>PS：如果把可变对象放到collection 之后又修改其内容，很容易会破坏set 的内部数据结构，使其失去固有的语义。

**2.尽量把对外公布出来的属性设为只读，而且只在确有必要时才将属性对外公布。**

**3.当我们想外部暴露只读属性、内部需要修改属性，这样子通常是在内部将readonly 属性重新声明为readwrite。但是如果该属性是nonatomic 的，这样子做可能会产生 “竞争条件”（rece condition）。在对象内部写入某属性时，对象外的观察者也许正在读取该属性。若想避免此问题，我们可以在必要时通过 “派发队列”（dispatch queue）等手段，将所有的数据存取操作都设为同步操作。**

**4.虽然属性对外设置成readonly 了，但是外部仍能通过 “键值编码”（Key-Value Coding，KVC）技术设置这些属性值。[object setValue:@"abc" forKey:@"name"] ，这样子可以修改name 这个属性，KVC 会在类中查找 “setName：” 方法来修改属性值。**

**5.还可以通过类型信息查询功能，查出属性所对应的实例变量在内存中的偏移量，从此来人为设置这个实例变量的值。**

**注意点总结：**

- 尽量创建不可变的对象。
- 若某属性仅可于对象内部修改，则在 “class-continuation 分类” 中将其由readonly 属性扩展成readwrite 属性。
- 不要把可变的collection 作为属性公开，而应提供相关方法，以此修改对象中的可变collection。

## 3.5 使用清晰而协调的命名方式

### 1、方法与变量命名

**方法和变量名使用 “驼峰式大小写命名法”：以小写字母开头，其后每个单词首字母大写。**

**1.方法名言简意赅，能准确表达方法功能，不易太长。**

**2. 如果方法的返回值是新创建的，那么方法名的首个词应该是返回值的类型**

```
- stringWithString

- intValue
```

**除非前面还有修饰语，例如localizedString。属性的存取方法不遵循这种命名方式。**

```
- localizedStringWithFormat:

- initWith
```

**3. 不要使用str这种简称，应该使用string 这样的全称。**

**4. Boolean 属性应加is前缀。如果某方法返回非属性的Boolean 值，那么应该根据其功能，选用has 或is 当前缀。**

**5. 将get 这个前缀留给那些借由 ”输出参数“ 来保存返回值的方法，比如说，把返回值填充到 ”C语言式数组“ 里的那种方法就可以使用这个词做前缀。**

### 2、类与协议的命名

**1. 类名也采用驼峰式命名法，不过其首字母需要大写，通常还会加三个以上前缀字母，避免命名空间冲突。**

**2. 命名应该协调一致，从其他框架继承子类，务必遵循其命名惯例。**
> UIView 子类末尾必须是View，委托协议末尾必须是Delegate。

**3. 起名时应遵从标准的Objective-C 命名规范，这样子创建出来的接口更容易为开发者所理解。**

## 3.6 为私有方法名加前缀

**1. 给私有方法的名称加上前缀，这样可以很容易地将其同公共方法区分开。**

> 一种方案是加前缀 p_

**2. 不要单用一个下划线做私有方法的前缀，因为这种做法是预留给苹果公司用的。**


### 3.7 理解Objective-C 错误模型

**1. ARC 默认不是 “异常安全的”，如果抛出异常，那么应在作用域末尾释放的对象现在却不会自动释放了。**

> 想要生成 “异常安全的” 代码，可以设置编译器的标志来实现 “-fobjc-arc-exceptions”。但是这样没有异常的情况也会执行这些代码。

**2. 平常很难写出在抛出异常时不会导致内存泄漏的代码，Objective-C 语言现在采用的办法是：只在极其罕见的情况下抛出异常，抛出异常应用程序直接退出，不考虑修复问题，不用再写复杂的 “异常安全” 代码。**

**3. 只有发生了可使整个应用程序崩溃的严重错误时，才应使用异常。**

**4. 在 “不那么严重的错误”，令方法返回nil/0，或者是使用NSError，表明其中有错误发生。**

> 创建对象时，可用 返回nil/0/-1

> 方法逻辑错误或者需要详细错误信息时，考虑 NSError

**5. NSError 可以经由此对象，把导致错误的原因回报给调用者。**

**NSError 可封装3条信息：**

- Error domain （错误范围，其类型为字符串）
- Error code （错误码，其类型为整数）
- User info （用户信息，其类型为字典）

**1）通过委托协议（delegate）来传递NSError**

```
- (void) connection:(NSURLConnection *)connection didFailWithError:(NSError *)error;
```

**2）经由方法的 “输出参数” 返回给调用者**

```
// 方法定义
// 返回 BOOL，容易判断（不需要查询 error 参数）。
// NSError ** 为指向指针（NSError *）的指针
- (BOOL) doSomething:(NSError **)error {
  if(/*there was an error*/){
   // error 参数不是nil,不然解引用异常
   if(error){
         // *error 解引用，即NSError *指针指向新对象。
         *error = [NSError errorWithDomain:domain
                                      code:code
                                  userInfo:userInfo];
           return NO;
     }
  }else{
      return YES;
  }
}

// 用法
NSError *error = nil; // 如果想获取 eroor 详情，不能为 nil
BOOL ret = [objecr doSomething:&error]
if(ret){
    //to do
}
```

## 3.9 理解NSCopying 协议

**1. 使用对象经常需要拷贝它，此操作通过copy 方法完成。如果想令自己的类支持拷贝操作，那就实现NSCopying 协议，该协议只有一个方法：**

```
- (id)copyWithZone:(NSZone *)zone
```

不必担心zone参数
 
> 以前开发程序，会把内存分成不同的 “区”（zone），而对象会创建在不同区里面，现在不用了，每个程序只有一个区：“默认区”（default zone）。因此不必担心 zone 参数

**2. NSMutableCopying 协议跟NSCopying 类似，也只有一个方法：**

```
- (id)mutableCopyWithZone:(NSZone *)zone
```

**3. 如果你的类分可变版本与不可变版本，这两个协议你都应该实现。**

**4. 注意：在可变对象上调用copy 方法返回另外一个不可变类的实例。**

```
- [NSMutablArray copy] => NSArray
- [NSArray mutableCopy] => NSmutableArray
```

**5. 在编写拷贝方法时，还要确定一个问题：应该执行 “深拷贝”（deep copy）还是 “浅拷贝”（shallow copy）。**

**6. 深拷贝是指在拷贝对象自身时，将其底层的数据也一并复制过去；浅拷贝只对拷贝对象的指针，并不会拷贝底层的数据。Foundation 框架中的所有collection 类默认都执行浅拷贝。**

**7. 没有专门定义深拷贝的协议，所以具体执行方式由每个类来确定。另外不要假设遵从了NSCopying 协议的对象都会执行深拷贝。绝大多数情况下，执行的都是浅拷贝。**

**8. 如果你所写的对象需要深拷贝，那么可以考虑新增一个专门执行深拷贝的方法。**

```
- (id)initWithSet:(NSArray *)array copyItems:(BOOL)copyItems
```































































































































































