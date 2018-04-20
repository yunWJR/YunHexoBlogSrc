---
title: iOS11 适配
date: 2018-04-17 16:13:34
categories: iOS
tags: [iOS,Objective-C]
---

# 1、启动页

如果启动页采用 Launch Imaged Sourc，则需要添加iPhoneX的启动图，不然整个 App 上下将有部分不能显示区域。

启动图的尺寸为：1125\*2436px 即：375\*812@3x

# 2、safeArea

iOS11为UIViewController和UIView增加了两个新的属性safeAreaInsets和safeAreaLayoutGuide, 通过这两个属性我们可以获得安全区域的范围, 
我们要做的是让那些不能被遮挡的内容和控件在安全区域范围内显示,

- safeAreaInsets 适用于手动计算.
- safeAreaLayoutGuide 适用于自动布局.

## 手动布局
新增方法，用于在 SafeArea 改变时，重新布局
- (void)viewSafeAreaInsetsDidChange;

# 3、Masonry

**由于引入了 safeArea，需要修改topLayoutGuide 和 bottomLayoutGuide。**

```
        // 对于
        make.top.equalTo(self.topLayoutGuide);
        make.bottom.equalTo(self.bottomLayoutGuide);
        
        // 可直接改为
        make.top.equalTo(self.view);
        make.bottom.equalTo(self.view);
```

**针对 iOS11,可以引入 safeArea**

```
        if (@available(iOS 11.0, *)) {
            // crash 
            // make.edges.equalTo(self.view.mas_safeAreaLayoutGuide);
            
            // error: self.view.mas_safeAreaLayoutGuide is self.view.mas_safeAreaLayoutGuideBottom

            //ok
            make.left.equalTo(self.view.mas_safeAreaLayoutGuideLeft);
            make.right.equalTo(self.view.mas_safeAreaLayoutGuideRight);
            make.top.equalTo(self.view.mas_safeAreaLayoutGuideTop);
            make.bottom.equalTo(self.view.mas_safeAreaLayoutGuideBottom);
        } else {
            make.edges.equalTo(self.view);
        }
```

**新增的 safeArea示例**

```
    [view1 mas_makeConstraints:^(MASConstraintMaker *make) {
        make.edges.equalTo(self.view.mas_safeArea).inset(10.0);
    }];
    
    [view2 mas_makeConstraints:^(MASConstraintMaker *make) {
        make.center.equalTo(self.view.mas_safeArea);
        make.width.height.equalTo(self.view.mas_safeArea).sizeOffset(CGSizeMake(- 40.0, - 40.0));
    }];
    
    [view3 mas_makeConstraints:^(MASConstraintMaker *make) {
        make.center.equalTo(self.view.mas_safeArea);
        make.width.equalTo(self.view.mas_safeArea).sizeOffset(CGSizeMake(- 60.0, - 60.0));
        make.height.equalTo(self.view.mas_safeArea).sizeOffset(CGSizeMake(- 60.0, - 60.0));
    }];
    
    [leftTopView mas_makeConstraints:^(MASConstraintMaker *make) {
        make.left.top.equalTo(self.view.mas_safeArea);
        make.width.height.equalTo(@(size));
    }];
    
    [rightTopView mas_makeConstraints:^(MASConstraintMaker *make) {
        make.right.equalTo(self.view.mas_safeAreaRight);
        make.top.equalTo(self.view.mas_safeAreaTop);
        make.width.height.equalTo(@(size));
    }];
    
    [leftBottomView mas_makeConstraints:^(MASConstraintMaker *make) {
        make.left.equalTo(self.view.mas_safeAreaLeft);
        make.bottom.equalTo(self.view.mas_safeAreaBottom);
        make.width.height.equalTo(@(size));
    }];
    
    [rightBottomView mas_makeConstraints:^(MASConstraintMaker *make) {
        make.right.bottom.equalTo(self.view.mas_safeArea);
        make.width.height.equalTo(@(size));
    }];
    
    [leftView mas_makeConstraints:^(MASConstraintMaker *make) {
        make.left.centerY.equalTo(self.view.mas_safeArea);
        make.width.height.equalTo(@(size));
    }];
    
    [rightView mas_makeConstraints:^(MASConstraintMaker *make) {
        make.right.equalTo(self.view.mas_safeAreaRight);
        make.centerY.equalTo(self.view.mas_safeAreaCenterY);
        make.width.height.equalTo(@(size));
    }];
    
    [topView mas_makeConstraints:^(MASConstraintMaker *make) {
        make.top.equalTo(self.view.mas_safeAreaTop);
        make.centerX.equalTo(self.view.mas_safeAreaCenterX);
        make.width.height.equalTo(@(size));
    }];
    
    [bottomView mas_makeConstraints:^(MASConstraintMaker *make) {
        make.bottom.centerX.equalTo(self.view.mas_safeArea);
        make.width.height.equalTo(@(size));
    }];
    
    [centerView mas_makeConstraints:^(MASConstraintMaker *make) {
        make.center.equalTo(self.view.mas_safeArea);
        make.width.height.equalTo(@(size));
    }];
```



# 4、prefersLargeTitles

该属性简单理解为**大标题**，当设置为 YES 时，导航栏高度将变为：96px，正常情况下为：44px；

应该如果要设置该属性，则需要在实例 Vc 中去获取导航栏的高度。

如：

```
self.navigationController.navigationBar.prefersLargeTitles = YES;
self.navigationItem.largeTitleDisplayMode = UINavigationItemLargeTitleDisplayModeAutomatic;
```

![](http://ot8psglzx.bkt.clouddn.com/1281817-20171120105905915-716853123.png?imageMogr2/thumbnail/!70p)

# 5、UITableView

UITableView莫名奇妙的偏移20pt或者64pt，

还有某些界面UITableView的sectionHeader、sectionFooter高度与设置不符的问题。

在iOS11中如果不实现

```
- tableView: viewForHeaderInSection: 
- tableView: viewForFooterInSection:
```

则 

```
- tableView: heightForHeaderInSection: 
- tableView: heightForFooterInSection:
```

不会被调用，导致它们都变成了默认高度。
这是因为tableView在iOS11默认使用Self-Sizing，UITableView的estimatedRowHeight、estimatedSectionHeaderHeight、 estimatedSectionFooterHeight三个高度估算属性由默认的0变成了UITableViewAutomaticDimension。

解决办法：

1. 实现对应的方法

2. 三个属性设为0。


