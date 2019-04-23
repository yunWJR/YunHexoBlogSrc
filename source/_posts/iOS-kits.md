---
title: 我的 iOS 框架简介
date: 2019-01-01 10:10:00
categories: iOS
tags: [iOS,Objective-C]
---

# 我的 iOS 框架简介

在平时 iOS 开发中，自己积累了一些功能框架，现在年前，正好有时间，把各个框架的功能和使用说明写了一下。希望能帮助到有用的人。

如果在使用中有什么BUG，疑问或者建议，都可以联系我：email:wangjr@mail.tsinghua.edu.cn 

## 1、YunBaseApp

[github](https://github.com/yunWJR/YunBaseApp)

自己开发的 iOS 应用开发的基本框架，涉及 App 中等各种功能：UIViewController 的封装、主题管理、帐号管理、日志管理、、加载页、提示信息、错误封装等。

主要模块：

- Account （用户信息管理）

- ActionListView （Action选择控件）

- AlertView （提示控件）

- Cache （缓存管理）

- Error （Error管理）

- HudView （HudView 基类）

- Log （日志封装）

- Rqt （网络请求封装）

- Theme （主题管理）

- View （UIView 和 UIViewController 的封装）

- ViewCategory (UIView 和 UIViewController的扩展)

## 2、YunKits

[github](https://github.com/yunWJR/YunKits)

iOS 基本库的一些封装扩展。主要包括：

- BaseView （对UIView、UIViewController、UITableView 的一些扩展）

- Categories （分类扩展）

- Factory （一些对象和控件的工厂方法）

- Macro （一些常用宏。建议少用宏，尽量用静态变量或者实例变量。）

- Tools （工具类）

## 3. YunImgView

[github](https://github.com/yunWJR/YunImgView)

封装的 iOS 图片列表控件，用 Objective-C 编写

该库主要包括两部分：1）YunImgView 图片列表库。2）YunSelectImgHelper 图片视频选择库。

## 4. YunQiniuHelper

[github](https://github.com/yunWJR/YunQiniuHelper)

自己封装的 iOS 端的七牛上传工具，使用 Objective-C。

可以上传单个文件，多个文件，指定 key。

## 5. YunWebView

[github](https://github.com/yunWJR/YunWebView)

使用Objective-C 实现的自定义 WebView，封装了 App 接口提供给 Web 前端使用，适用于 App 内嵌功能网页。

## 6. YunImageBrowser

[github](https://github.com/yunWJR/YunImageBrowser)

基于MWPhotoBrowser，进行修改的图片浏览控件

* 修改依赖库，支持最新的 SDWebImage
* 修改样式，支持 iPhone X
* 修复一些 BUG








