---
title: Objective-C 属性
date: 2018-04-16 09:05:48
categories: iOS
tags: [iOS,Objective-C]
---

# Objective-C 属性(property)

### @property

用于声明属性，自动实现属性的读写方法。

## 属性特质

原子性、读写权限、内存管理语义、方法名、其他。

### 1、原子性

- atomic -默认

  > 占用部分资源、效率一般、线程安全

- nonatomic 

  > 非原子、效率高、线程不安全

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
  > 
  > 赋值时，保留新值，新值引用计数+1，释放旧值（引用计数-1）。
  > 
  > 用于所有的实例变量和局部变量、其他常规对象引用。
  > 
  > 注意：可变对象应该使用strong，如NSMultiString

- copy 

  > 复制、用于引用类型
  > 
  > 赋值时，拷贝新值（新对象引用计数为1），释放旧值（引用计数-1），不改变新值（引用计数不变）。
  > 
  > copy的本质为复制该内存所存储的内容，重新创建一个对象赋给其相同的内容，对于实现了NSCopying协议的对象有效。
  > 
  > 用于不可变对象：NSString、block、NSArray、NSDictionary等
  > 
  > 注意：用于可变对象时，设置值后，变为不可变对象

- weak 

  > 弱引用、用于引用类型
  > 
  > 赋值时、单纯的引用新对象地址，不改变新对象（引用计数不变），不改变旧对象（引用计数不变）
  > 
  > 当引用对象释放后，其值置为nil

- unsafe_unretained 

  > 类似assign、适用于引用类型、不安全的弱引用
  > 
  > 功能类似于weak、对象摧毁后，不置nil、不安全，可用weak代替

### 4、方法名

- getter=<name>

  > ```
  >   @property (nonatomic, getter=isOn) BOOL on;
  > ```

- setter=<name>

  > ```
  >   @property (nonatomic, setter=setOnState) BOOL on;
  > ```

### 5、其他

nonnull, null_resettable, nullable
