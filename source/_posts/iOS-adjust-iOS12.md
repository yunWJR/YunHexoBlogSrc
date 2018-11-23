---
title: iOS12 XCode10 适配
date: 2018-11-03 17:22:31
categories: iOS
tags: [iOS,Objective-C]
---

# iOS12 XCode10 适配

## 1. libstdc++弃用  报错Undefined symbols

 XCode10编译报错`ndefined symbols for architecture XXX`，如果你的工程中有libstdc++依赖（可从Linked Frameworks and Libraries 项查看），那么就会出现这类错误。

因为苹果在XCode10和iOS12中移除了libstdc++这个库，由libc++这个库取而代之，苹果的解释是libstdc++已经标记为废弃有5年了，建议大家使用经过了llvm优化过并且全面支持C++11的libc++库。

> libstdc++.dylib是C++98版本的标准库实现动态库，而libc++.dylib是C++11版本的标准库实现动态库。libc++是一个更加新的C++标准库实现，它完全支持C++11标准。因此苹果弃用了libstdc++.dylib，这符合苹果一贯的作风。

### 解决方法

* **最直接的是修改依赖库，支持libc++.dylib**

* **临时方法**

	将libstdc++.dylib拷贝到 XCode中，共四个地方
	
	[libstdc++.dylib下载地址](http://qnyuntmp.yunsoho.cn/libstdc++.zip)

```
sudo cp CoreSimulator/* /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/Library/CoreSimulator/Profiles/Runtimes/iOS.simruntime/Contents/Resources/RuntimeRoot/usr/lib/
sudo cp MacOSX/* /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk/usr/lib/
sudo cp iPhoneOS/* /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS.sdk/usr/lib/
sudo cp iPhoneSimulator/* /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneSimulator.platform/Developer/SDKs/iPhoneSimulator.sdk/usr/lib/
```

## 2. UICollectionViewCell 高度计算不正确

**更新 iOS12后，一定要检查所有用到UICollectionViewCell的界面，因为UICollectionViewCell可能出现高度计算不正确的现象。**

iOS12对AutoLayout做出了性能优化，但是更新 iOS12后，发现一些UICollectionViewCell的高度不正确，一时间也调试不出什么问题，因此就采用手动计算高度暂时解决。

这里有一篇同样的问题，解决思路可供参考[链接](http://www.cocoachina.com/ios/20181023/25267.html)

### 解决方法

* **1. 手动计算高度**

* **2. 忽略 contentView，直接把 subView 加到 cell 上**


## 3. StatusBar 网络状态

如果app通过状态栏的网络状态指示器去判断手机当前联网状态，修改进行修改，因为iOS12 更改了StatusBar内部结构。

[参考链接](https://blog.csdn.net/wxs0124/article/details/80613847)

```
+ (NSString *)getIphoneXNetWorkStates {    
    UIApplication *app = [UIApplication sharedApplication];
    id statusBar = [[app valueForKeyPath:@"statusBar"] valueForKeyPath:@"statusBar"];
    id one = [statusBar valueForKeyPath:@"regions"];
    id two = [one valueForKeyPath:@"trailing"];
    NSArray *three = [two valueForKeyPath:@"displayItems"];
    NSString *state = @"无网络";
    for (UIView *view in three) {
        //alert: iOS12.0 情况下identifier的变成了类"_UIStatusBarIdentifier"而不是NSString，所以会在调用“isEqualToString”方法时发生crash，
        //修改前
//        NSString *identifier = [view valueForKeyPath:@"identifier"];
        //修改后
        NSString *identifier = [[view valueForKeyPath:@"identifier"] description];
        if ([identifier isEqualToString:@"_UIStatusBarWifiItem.signalStrengthDisplayIdentifier"]) {
            id item = [view valueForKeyPath:@"_item"];

            //alert: 这个问题和上边一样itemId是_UIStatusBarIdentifier 类型，不是string
            NSString *itemId = [[item valueForKeyPath:@"identifier"] description];
            if ([itemId isEqualToString:@"_UIStatusBarWifiItem"]) {
                state = @"WIFI";
            }
            state = @"不确定";

        } else if ([identifier isEqualToString:@"_UIStatusBarCellularItem.typeDisplayIdentifier"]) {
            UIView *statusBarStringView = [view valueForKeyPath:@"_view"];
            // 4G/3G/E
            state = [statusBarStringView valueForKeyPath:@"text"];
        }

    }

    return state;
}
```

## iOS12新功能

### 1. 刘海屏判断

```
#define isNotchMobile ([UIScreen instancesRespondToSelector:@selector(currentMode)] ? (CGSizeEqualToSize(CGSizeMake(1125, 2436), [[UIScreen mainScreen] currentMode].size)||CGSizeEqualToSize(CGSizeMake(1242, 2688), [[UIScreen mainScreen] currentMode].size)||CGSizeEqualToSize(CGSizeMake(828, 1792), [[UIScreen mainScreen] currentMode].size)) : NO)
```



















