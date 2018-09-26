---
title: Hackintosh 黑苹果总结
date: 2018-09-13 18:22:42
categories: Hackintosh
tags: [Hackintosh,黑苹果]
---

# Hackintosh 黑苹果总结

> 个人用黑苹果开发iOS、java已经有2年多。之所以选择黑苹果，还是因为 Macbook 性能太弱，而且发热严重。2年之久，基本随着 Mac系统更新，也会更新系统（不然无法更新最新的Xcode，纯 java 开发倒是无所谓）。只有一次打开【文件保险箱】功能挂了（下面有说明，千万不要打开该选项），其他时候都没问题。分享其中的一些经验，给愿意尝试黑苹果的朋友。

* 个人黑苹果电脑的配置：

	电脑一：台式机（i7-87000k 华擎 Z370M-ITX 16G GT750ti）

	电脑二：台式机（i5-4590 华硕 b85m 8G）

	电脑三：联想 Y700笔记本（i5-6300 8G GT960M）

* 帮人配的电脑：

	电脑一：台式机（i7-77000 华擎 deskmini 16G）

	电脑二：台式机（i5-7500 华硕 B150 8G）

    [EIF 分享-包括以上提到的几种配置机型](https://github.com/yunWJR/Hackintosh_List)

## 一、哪些电脑可以安装黑苹果

### 1、笔记本
	
如何确定笔记本可以安装：
	 
1）搜索笔记本型号，看有安装成功的案例没。
	
2）看CPU，如果 CPU 的型号与苹果已经发布的笔记本相同或类似（类似定义为同代 CPU），那么 CPU 应该没问题。

3）看显卡：如果笔记本有独显，那么独显基本是不能使用的，只能使用集显。偶尔少数笔记本有独显，主板不能屏蔽独显，导致无法安装。

4）以上满足的话，可以尝试安装。

### 2、台式机

如何确定台式机可以安装：
	 
1）搜索台式机配置，看有安装成功的案例没。
	
2）看CPU，如果CPU的型号与苹果已经发布的笔记本相同或类似（类似定义为同代 CPU），那么 CPU 应该没问题，基本都常见的 intel 台式机 CPU 都可以安装。最新的 AMD 都 CPU 也有大神放出内核，可以安装。

3）看显卡：如果有独显，比较新的 AMD 显卡都可以支持，N 卡一般也支持，有 WebDriver。

4）主板：一般都支持，技嘉的一般支持原生电源管理，比较好。华擎的支持比较到位，曾经几块主板专门出过安装黑苹果的 BIOS，良心。

5）网卡：一般 intel 的有线网卡都支持，无线的话，选择 苹果采用的型号，容易驱动。

### 3、建议

1）如果买新电脑

笔记本可以买没有独显的，因为有也一版用不上。先查下哪些比较好安装的机型，照着买就行。

台式机可以参考 [tonymacx86](https://www.tonymacx86.com/buyersguide/building-a-customac-hackintosh-the-ultimate-buyers-guide/)上的配置，都很容易安装。

2）最好配2块以上硬盘。

一块安装 Mac。另外一块安装 Windows，或者作为备份盘（TimeMachine）。如果作为生成环境使用，建议单独配置一块硬盘作为 TimeMachine 的备份盘。

3）如果要独显，最好选 AMD 的卡，可以很好的原生驱动。

4）如果配无线，最好选 Mac 电脑上用过的型号，容易驱动。



## 二、安装流程


### 最简方式：

1、Mac 上 AppStore 中下载 Mac 系统

2、制作安装镜像到 U 盘，可以借助工具 [DiskMaker](http://diskmakerx.com/)

3、制作 EFI 启动分区，可以制作在 U 盘上，也可以制作在硬盘上。

4、放入 EFI 分区启动文件（kext 和 config 配置文件等）

5、设置好 BIOS 选项

6、从 EFI 启动安装

7、完善安装（各硬件驱动）

8、完善 EFI，可将稳定的 EIF 文件，放入 Mac 分区的 EFI 分区。从 Mac 分区启动。

### 注意事项

1、不要随意升级 Mac 新版本，可能造成 kext 不兼容。

2、最好配一块备份盘，用 TimeMachine 备份。TimeMachine 确实好用。

3、一般不要去动 SLE 下的系统kext，也尽量不要把 kext 放入 SLE 下面，补丁 kext 都可以放在 EFI 下的 kext 中调试。

4、千万不要打开『安全与隐私』中的 【**文件保险箱**】功能，该功能与硬件相关，打开后，黑苹果就GG。

5、config 中的硬件 ID 尽量用同一个（同一台机子），新机子第一次安装时，随机生成一个。频繁更改 ID，会让你重新登录 AppleId。

### 可以参考的网站

[tonymacx86 - 国外很活跃度黑苹果网站，还有配置推荐](https://www.tonymacx86.com/)

[pcbeta - 国内活跃度黑苹果网站](http://bbs.pcbeta.com/forum.php?mod=forumdisplay&fid=558&filter=author&orderby=dateline)

[cloverefiboot - clover项目现在地址](https://sourceforge.net/projects/cloverefiboot/)

[RehabMan bitbucket -  RehabMan的 kext下载](https://bitbucket.org/RehabMan/)

[RehabMan git - RehabMan的 kext下载和一些配置信息](https://github.com/RehabMan)

[acidanthera git - 作者写了很多有用的 kext](https://github.com/acidanthera)







