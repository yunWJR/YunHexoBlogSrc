---
title: 发布线上安装iOS应用（ipa）
date: 2018-05-21 17:38:48
categories: iOS
tags: [iOS,Objective-C,总结]
---

# 发布线上安装iOS应用（ipa）

发布 ipa 文件到线上，通过itms-services 协议访问安装。

### 一、适用对象

#### 1. 企业证书，直接安装

> 所有人都可以下载安装，用户打开前，需要信任手动信任企业证书。

#### 2. 个人/公司证书，给添加过 UDID 的设备安装

> 适用于部分设备安装（添加了 UDID 的设备）

### 二、操作步骤

本人采用Xcode 生成安装文件，步骤如下：

#### 1. 打开Xcode，的 Organizer，选择需要发布的应用
 
#### 2. 点击 Export，选择 Ad Hoc，点击 Next

#### 3. 勾选如图选项：点击 Next

![](http://ot8psglzx.bkt.clouddn.com/WX20180521-170859.png?imageMogr2/thumbnail/!70p)

#### 4. 填写相关项，需要 https 协议的链接，可用 github 仓库。

![](http://ot8psglzx.bkt.clouddn.com/WX20180521-171144.png?imageMogr2/thumbnail/!70p)

#### 5. 根据情况执行后续操作，得到manifest.plist文件。上传到服务器，可选择 git。

#### 6. 生成下载地址：如：

```
itms-services://?action=download-manifest&url=https://raw.githubusercontent.com/xxx/dis_ipa/master/manifest.plist
```

#### 7. 手机上用 Safari 打开以上链接，进行安装。
