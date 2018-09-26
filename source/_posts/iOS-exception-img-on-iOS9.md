---
title: iOS9 因为图片莫名闪退
date: 2018-05-21 17:40:47
categories: iOS
tags: [iOS,Objective-C,异常]
---

# iOS9 因为图片莫名闪退

> 一般错误信息与图片相关，如

~~~
-[CUIStrucTuredThemeStore renditionWithKey:usingKeySignature:] 
~~~

> 仅在 iOS9及一下系统出现

### 问题说明

ios9.3以下系统不支持非RGB 色域的图片，需要排查所有图片。

### 排查步骤


#### 1. 获取ipa 文件

1. 直接从 Xcode 导出

2. 从 iTuns 下载（没找到在哪儿）

#### 2. 获取Assets.car文件

1. 解压ipa, 找到 Payload 中的 .app 文件, 显示包内容，找到 Assets.car 文件，拷贝到工作目录。

#### 3. 获取asset.json文件

```
cd 工作目录

sudo xcrun --sdk iphoneos assetutil --info ./Assets.car > asset.json

输入密码，生成asset.json文件
```

#### 4. 查找非 RBG图片

查询 asset.json文件中的"DisplayGamut" : "P3"，即为不能用的图片

### 补充
在查阅资料时发现, 很多资料都提到过在项目中运行一个脚本将P3图片进行转换, 由于此种方法朕没有实际验证过, 所以只做个摘录

```
#!/bin/bash DIRECTORY=$1 echo "----Passed Resources with xcassets folder argument is <$DIRECTORY>" echo "----Processing asset:"

find "$DIRECTORY" -name '*png' -print0 | while read -d $'\0' file; do echo "---------$file" sips -m "/System/Library/Colorsync/Profiles/sRGB Profile.icc" "$file" --out "$file" done

echo "----script successfully finished"

```