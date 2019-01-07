---
title: EffectiveObjective-C2.0 笔记 - 第六部分
date: 2018-10-07 21:11:45
categories: iOS
tags: [iOS,Effective Objective-C 2.0]
---

**EffectiveObjective-C2.0 笔记 - 第六部分**


# 6. 块与大中枢派发

## 6.1 理解 “块” 这一概念

**一、块的基础知识**

块是C、C++、Objective-C 中的词法闭包。

**1. 块用 “^” 符号来表示，后面跟着一对花括号，括号里面是块的实现代码。**

> 块其实就是个值，与 int，flout，Objective-C对象一样，而且自有其相关类型，可以赋值给变量；块类型的语法和函数指针类似。

   ```
   ^{
   	//block implementation herer	
   }

   //块类型的语法结构如下
   //return_type (^block_name)(parameters)
   void (^oneBlock)() = ^{
   	//block implementation herer	
   }
   ```

**2. 在声明块的范围内，所有变量都可以被其捕获。**

> 默认情况下被块捕获的变量是不可以在块里修改的，不过可以在声明变量的时候加上__block 修饰符，这样子就可以在块内修改了。

> 如果块所捕获的变量是对象类型，那么就会自动保留它，在系统释放这个块的时候，也会将其一并释放。

> 块总能修改实例变量，所以在声明时无须加__block。不过如果通过读取或写入操作捕获了实例变量，那么也会自动把self 变量一并捕获了，因为实例变量是与self 所指代的实例关联在一起的。

> 如果 self 所指代的那个对象同时也保留了块，那么这种情况通常就会导致"保留环"。

**二、块的内部结构**

**1. 块本身也是对象，在存放块对象的内存区域中，首个变量是指向Class 对象的指针(isa 指针)。**

   ![](http://qnyunyun.yunsoho.cn/adadsa.png)

**2. invoke 变量是这个函数指针，指向块的实现代码。**

> 函数原型至少要接受一个void* 型的参数，此参数代表块。为什么要把块对象作为参数传进来呢，因为在执行块的时候，要从内存中把这些捕获到的变量读出来。

> descriptor 变量是指向结构体的指针，这个结构体包含块的一些信息。

**三、全局块、栈块及堆块**

**1. 定义块的时候，其所占的内存区域是分配在栈中，意思就是，块只在定义它的那个范围内有效。**

   ```
   void (^block)();
   if(***){
   	block = ^(){
   		NSLog(@"Block A");
   	};
   }else{
   	block = ^(){
   		NSLog(@"Block B");
   	};
   }
   block();
   ```
   
- 栈块

> 定义在if else 语句中的两个块都分配在栈内存中，编译器会给每个块分配好栈内存，然而等离开了相应的范围之后，编译器有可能把分配给块内存覆写掉。所以这里执行block() 有危险。

- 堆块

> 为了解决这个问题，可以给块发送copy 消息以拷贝之。这样子的话，就可以把块从栈复制到堆可。一旦复制到堆上，块就成了带引用计数的对象了，后续的复制操作都不会真的执行复制，只是递增块对象的引用计数。

- 全局块

> 全局块声明在全局内存里，而且也不能被系统回收，相当于单例。由于运行该块所需的全部信息在编译期确定，所以可以把它作为全局块，这是一种优化技术：若把如此简单的块当成复杂的块来处理，那就会在复制及丢弃该块时执行一些无谓的操作。

   ```
   void (^block)() = ^(){
   		NSLog(@"Block A");
   	};
   ```


## 6.2 为常用的块类型创建 typedef

**1. 每个块都具备其 “固定类型”，因而可将其赋值给适当类型的变量。**

**2. 由于块类型的语法比较复杂难记，我们可以给块类型起个别名。**

> 用C 语言中的 “类型定义” 的特性。typedef 关键字用于给类型起个易读的别名。

```
   typedef int(^EOCSomeBlock)(BOOL flag, int value);

   EOCSomeBlock block = ^(BOOL flag, int value){
   	//to do
   };
```

## 6.3 块降低代码分散程度

**场景：**

> 异步方法执行完任务，需要以某种手段通知相关代码。经常使用的技巧是设计一个委托协议，令关注此事件的对象遵从该协议，对象成了delegate 之后，就可以在相关事件发生时得到通知了。

**使用块来写的话，代码会更清晰，使得代码更加紧致。**


## 6.4 用块引用其所属对象时不要出现保留环

**1. 如果块所捕获的对象直接或间接地保留了块本身，那么就得当心保留环问题。**

**2. 一定要找个合适的时机解除保留环，而不能把责任推给API的调用者。**

## 6.5 多用派发队列，少用同步锁

**1. 同步锁**

> 如果有多个线程要执行同一份代码，那么有时可能会出问题，这种情况下，通常要使用锁来实现某种同步机制。在GCD 出现之前，有两种办法：

   * 采用内置的 “同步块”（synchronization block）

     ```
     - (void)synchronizedMethod {
     	@synchronized(self){
     		//safe
     	}
     }

     /*
     这种写法会根据给定的对象，自动创建一个锁，并等待块中的代码执行完毕，执行到代码结尾，锁就释放了。

     但是，滥用 @synchronized(self) 则会降低代码效率，因为共用同一个锁的那些同步块，都必须按顺序执行。
     */
     ```

   * 直接使用NSLock 对象，也可以使用NSRecursiveLock “递归锁”，线程能多次持有该锁，而且不会出现死锁。

     ```
     _lock = [[NSLock alloc] init];

     - (void)synchronizedMethod {
     	[_lock lock];
     	//safe
     	[_lock unlock];
     }
     ```

**2. 对于上面两种方法，有些缺陷，同步块会导致死锁，直接使用锁对象，遇到死锁，就会非常麻烦。**

**3. GCD以更简单、更高效的形式为代码加锁。**

**例子：**

> 属性是开发者经常需要同步的地方，可以使用atomic 特质来修饰属性，来保证其原子性，每次肯定可以从中获取到有效值，然而在同一个线程上多次调用获取方法(getter)，每次获取到结果未必相同，在两次访问操作之间，其他线程可能会写入新的属性值。

   使用 “串行同步队列”，将读取操作及写入操作都安排在同一个队列里，即可保证数据同步。

   ```
   _syncQueue = dispatch_queue_create("com.yun.syncQueue", NULL);

   - (NSString *)name {
       __block NSString *rstName;
       dispatch_sync(_syncQueue, ^{
           rstName = _name;
       });
       return rstName;
   }

   - (void)setName:(NSString *)name {
       dispatch_sync(_syncQueue, ^{
           _name = name;
       });
   }
   ```

> 上面是用串行同步队列来保证数据同步：把设置操作与获取操作都安排在序列化的串行同步队列里执行，这样子，所有针对属性的访问操作都是同步的了。
   
> 进一步优化，设置方法不一定非得是同步的，因为不需要返回值。这样子可以提高设置方法的执行速度，而读取操作与写入操作依然会按照顺序执行。

**优化：多个获取方法可以并发执行，而获取方法与设置方法不能并发执行。**


   我们还可以使用并发队列来实现,现在都是在并发队列上面执行任务，但是顺序不能控制，我们可以用栅栏(barrier)来解决。

   这两个函数可以向队列派发块，将其作为栅栏来使用：
  
```
   dispatch_barrier_sync(dispatch_queue_t queue,^(void)block)
   dispatch_barrier_async(dispatch_queue_t queue,^(void)block)
```
  
   在队列中，栅栏块必须单独执行，不能与其他块并行，这只对并发队列有意义，因为串行队列中的块总是按照顺序逐个执行的。并发队列如果发现接下来要处理的块是栅栏块，那么就一直要等到当前所有的并发块都执行完毕，才会单独执行这个栅栏块。执行完栅栏块，再按照正常方式向下处理。
   */

```
   -----> 现在并发队列 还不能满足要求
   _syncQueue1 = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
   - (NSString *)name {
       __block NSString * rstName;
       dispatch_sync(_syncQueue1, ^{
           rstName = _name;
       });
       return rstName;
   }

   - (void)setName:(NSString *)name {
       dispatch_async(_syncQueue1, ^{
           _name = name;
       });
   }

   -----> 转换写法 用栅栏块控制属性的设置方法 不能并行
   _syncQueue1 = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
   - (NSString *)name {
       __block NSString * rstName;
       dispatch_sync(_syncQueue1, ^{
           rstName = _name;
       });
       return rstName;
   }

   - (void)setName:(NSString *)name {
       dispatch_barrier_async(_syncQueue1, ^{
           _name = name;
       });
   }
```

## 6.6 多用GCD，少用performSelector 方法

**建议用GCD 替代performSelector，对于performSelector 遇到的问题，我们都可以用GCD 解决**

1. performSelector 可以任意调用方法，还可以延迟调用，还可以指定运行方法所用的线程，这些由 Objective-C 的动态性决定。

2. 但是如果是动态来调用performSelector 方法的时候，编译器都不知道执行的选择子是什么，必须到了运行期才能确定，这种情况在ARC下会报警告，因为编译器不知道方法名，所以不能运用ARC内存管理规则来判定返回值是否应该释放，对于这种情况ARC不会帮我们添加任何释放操作。

3. performSelector 方法调用的时候对于返回类型只能是void或对象类型，对于有返回值的需要自己做多次转换，对于参数的也最多只能传2个，介于此performSelector 还是比较不方便的。

4. 如果想把任务放在另一个线程上执行，那么最好不要用performSeletor  系列方法，而是应该把任务封装到块里，然后调用大中枢派发机制的相关方法来实现。

## 6.7 掌握GCD 及操作队列的使用时机

**1. 使用NSOperation执行后台任务**

- GCD在很多地方特别优秀，但是在执行后台任务时，GCD不一定是最佳方式，还有一种技术叫做NSOperationQueue，开发者可以把操作以NSOperation子类的形式放在队列中，而这些操作也可以并发执行。

- GCD是纯C的API，操作队列的则是Objective-C的对象。用NSOperationQueue类的“addOperationWithBlock” 方法搭配NSBlockOperation类操作队列，其语法与纯GCD方式非常类似。使用NSOperation及NSOperationQueue 的好处如下：
   
**1) 取消某个操作。**

> 如果使用操作队列，那么想取消操作是很容易的。运行任务之前，可以在NSOperation 对象调用cancel 方法，该方法会设置对象内的标识位，用以表明此任务不需执行，不过，已经启动的任务无法取消。若不是操作队列，而是把块安排到GCD 队列，那就无法取消了。那套架构是 “安排好任务之后就不管了”。开发者可以在应用层自己来实现取消功能，不过这样子做需要编写很多代码，而那些代码其实已经由操作队列实现好了。

**2. 指定操作间的依赖关系。**
   
> 一个操作可以依赖其他多个操作。开发者能够指定操作之间的依赖关系，使特定的操作必须在另外一个操作顺序执行完毕方可执行，比方说，从服务器下载并处理文件的动作，可以用操作来表示，而在处理其他文件之前，必须先下载 “清单文件”。后续的下载操作，都要依赖于先下载清单文件这一操作。如果操作队列允许并发的话，那么后续的多个下载操作就可以同时执行，但前提是它们所依赖的那个清单文件下载操作已经执行完毕。

**3. 通过键值观测机制监控NSOperation对象的属性。**

> NSOperation 对象有许多属性都适合通过键值观测机制（KVO）来监听，比如可以通过isCancalled 属性来判断任务是否取消。如果想在某个任务变更期状态时得到通知，或是想用比GCD 更为精细的方式来控制所要执行的任务，那么键值观测机制会很有用。

**4. 制定操作的优先级。**

> 操作的优先级表示此操作与队列其他操作之间的优先关系。优先级高的操作先执行，优先级低的后执行。操作队列的调度算法已经比较成熟。反之，GCD 则没有直接实现此功能的办法，GCD 的队列有优先级，但是是针对整个队列来说的，而不是针对每个块来说的。对于优先级这一点，操作队列所提供的功能比GCD 更为便利。

**5. 重用NSOperation 对象。**

> 系统内置类一些NSOperation 的子类供开发者调用，要是不想用这些固有子类的话，那就得自己来创建了。这些类就是普通的Objective-C 对象，能够存放任何信息。对象在执行时可以充分利用存于其中的信息，而且还可以随意调用定义在类中的方法。这比派发队列中哪些简单的块要强大。这些NSOperation 类可以在代码中多次使用。

## 6.8 通过Dispatch Group机制，根据系统资源状况来执行任务

**1. 一系列任务可归入一个dispatch group之中。开发者可以在这组任务执行完毕时获得通知。**

**2. 通过dispatch group，可以在并发式派发队列里同时执行多项任务。此时GCD会根据系统资源状况来调度这些并发执行的任务。开发者若自己实现此功能，则需编写大量代码。**

## 6.9 使用dispatch_once来执行只需运行一次的线程安全代码

1. 只需执行一次的线程安全代码时，使用dispatch_once。

> 经常需要编写 “只需执行一次的线程安全代码”。通常使用GCD 所提供的dispatch_once 函数，很容易就能实现此功能。

> 使用dispatch_once 可以简化代码，并且彻底保证线程安全

2.  单例示例

> 对于单例我们创建唯一实例，之前都是用@synchronized 加锁来解决多线程的问题，GCD 提供了一个更加简单的方法来实现。	

```objective-c
//单例
+(instancetype)shareInstance{
    static EOCClass *shareInstance;
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        shareInstance = [EOCClass new];
    });
    return shareInstance;
}
```

## 6.10 不要使用dispatch_get_current_queue 

1.  dispatch_get_current_queue 函数的行为常常与开发者所预期的不同。此函数已经废弃(iOS 6.0起废除)，只应做调试之用。

2. 由于派发队列是按层级来组织的，所以无法单用某个队列对象来描述 “当前队列” 这一概念。

3. dispatch_get_current_queue 函数用于解决由不可重入的代码引发的死锁，然而能用此函数的解决的问题，通常也能改用 “队列特定数据” 来解决。















































































































































