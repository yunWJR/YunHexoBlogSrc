---
title: iOS中的循环引用
date: 2018-04-20 09:17:29
categories: iOS
tags: [iOS,Objective-C,Sol]
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
