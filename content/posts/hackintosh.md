---
title: "黑果折腾笔记 WIP"
date: 2020-12-29T09:25:49+08:00
draft: false
tags: ["macOS"]
categories: ["macOS相关就搁这儿吧"]
featured_image: ""
description: ""
---
## 概述

组黑苹果有两个层次，如下所述：

1. 能用的黑果 ✅

   正确配置 OC 引导，能够正常的进入 macOS，能够安装并使用工作常用的软件。本人系程序猿一枚，常用的工作环境以及软件有 homebrew、golang、docker、vscode 等，这些软件因人而异。

2. 好用的黑果

   支持蓝牙、随航、隔空投送、正常休眠唤醒、各种 usb 设备正常工作。但这些都是使用黑果的非必要条件。

对于我个人来说，折腾黑果的最大动力就是经济效益。

我的需求是：

黑果能够提供给我一个**稳定有效**的工作环境。我不太介意每次开机需要重新插拔一下键盘或者鼠标的 USB 接口，也不太介意能否正常休眠唤醒（PS. 因为是台式机，所以不太需要休眠，能锁屏就行）。所以对我来说，第一目标就可以满足我的需求，第二目标中的一些功能可以等到需要时再慢慢想办法支持。

## 介绍

我最开始接触并动手组建自己的黑果是在 2020 年，这是一个尴尬的时间节点。因为在 2020 年，苹果公司推出了基于 arm 架构的 m1 芯片（据坊间测评，性能提升还蛮多）。这意味着基于x86的黑果可能没有多少日子了。但是，如果是为了工作或者单纯体验一下苹果系统，这并不是很大的问题。

目前，我已成功在两台不同配置的台式机上装上了macOS，分别是：

1. 8700k + 华硕ROG的m10f（z370芯片组）
2. 10700 + 微星迫击炮b460m

其中配置1是2018年组装的，并非以装黑果为目的，安装的时候遇到了一些小问题；而配置2是根据黑果的要求进行的硬件选购，安装起来要方便很多。

对于不同配置，安装过程大同小异，下面就来说说安装黑果的一个大致流程👇。

## 安装的流程步骤

安装大致分为以下5步：

1. 查看（确定）电脑配置
2. 收集（制作）EFI
3. 制作USB启动盘
4. 安装调试
5. mac setup

## 配置

我将以 8700k 这套为代表来对黑果硬件进行说明。

这套攒于2018年，是当时的顶配。目前服役2年多，当时主要是为了玩游戏，并未考虑后面会安装黑苹果。

先列下配置，再说下这套配置的问题，最后在说下配置的选择。

**配置单**

| 配件        | 型号                                                         |
| ----------- | ------------------------------------------------------------ |
| CPU         | Intel(R) Core(TM) i7-8700K CPU @ 3.70GHz                     |
| GPU（显卡） | NVIDIA GeForce GTX 1080 Ti<br />intel UHD630                 |
| 主板芯片组  | Z370                                                         |
| 声卡        | NVIDIA GP102 HDMI/DP @ NVIDIA GP102 - High Definition Audio Controller<br />Realtek ALC1220 @ Intel Kaby Point PCH - High Definition Audio Controller (Audio, Voice, Speech) |
| 网卡        | Intel Ethernet Connection (2) I219-V<br />Realtek RTL8822BE Wireless LAN 802.11ac PCI-E Network Adapter |
| 硬盘        | Samsung SSD 960 EVO 250GB<br />WDC WD20EZRZ-00Z5HB0          |

**配置问题**

这套配置中NVIDIA显卡和Realtek无线网卡是没法驱动的。

对于显卡而言，如果有剪辑视频需求，那么需要另行购入免驱A卡，N卡都是不能驱动的（只能安装较低版本的macOS）。本人主要是用来作后端开发，直接使用8700k自带uhd630核显就可以。

对于无线网卡，本人另外购买了博通的BCM94360CD，这是免驱卡，可以同时解决无线和蓝牙的问题。

**配置怎么选**

网上有很多类似的说明，下面是我的简要心得：

1. cpu选intel的，最好是带核显，amd虽然也行，但是安装起来比较折腾
2. 显卡，最好购买a卡，买之前先查查能否在macOS中免驱，N卡就别想了
3. 无线网卡，买博通的免驱网卡，在windows上也可以用
4. 如果是从零组机，那就照着别人的配置单抄吧。那么，整个安装过程就简单百倍。

其他的好像也没啥特别注意的地方了。如果想详细了解硬件限制，推荐阅读下面这篇文章：[Hardware Limitations](https://dortania.github.io/OpenCore-Install-Guide/macos-limits.html)。

## 制作EFI

最简单的方法就是收集别人做好的EFI。可以到github上以`CPU 芯片组 EFI`为关键词进行搜索，如`8700k z370 EFI`，也可以去远景论坛看看。

但是，我还是推荐自己动手制作EFI，其实也没那么困难。推荐的阅读材料《[OpenCore-Install-Guide](https://dortania.github.io/OpenCore-Install-Guide/)》。对于手动制作EFI的人来说，大概都会看这个install guide吧，我反正是看了两遍。

制作EFI，主要分为三大部分：

1. 制作USB启动盘
2. 收集Kexts、Drivers以及SSDT
3. 配置config.plist

👇是一些我认为的**关键术语**，以及我个人的理解：

| 术语        | 描述                                                         |
| ----------- | ------------------------------------------------------------ |
| **ACPI**    | 高级配置与电源接口。是一种规范，用来定义固件抽象接口。详细可以阅读 [维基百科](https://en.wikipedia.org/wiki/Advanced_Configuration_and_Power_Interface) |
| **Kexts**   | 内核拓展，可以理解为macOS的驱动                              |
| **Drivers** | 指UEFI驱动                                                   |
| **DSDT**    | Differentiated System Description Table. DSDT可以被看作是持有大部分信息的主体，就如同项目蓝图。 |
| **SSDT**    | Secondary System Description Table. SSDT传递较小的信息，可以想象为项目进展中的便利贴。 |

### 制作USB启动盘

1. 下载macOS安装镜像

   我使用过两种方法来下载macOS的安装镜像，根据实际的安装体验，我比较推荐recovery模式，更方便快捷:

   1. 使用recovery模式

      对于recovery模式，只需要在命令行执行一条简单的命令就可以了：

      ```bash
      ./macrecovery.py -b Mac-E43C1C25D4880AD6 -m 00000000000000000 -os latest download
      ```

      macrecovery是OC官方提供的工具，其通过下载一个恢复dmg，在安装时再次联网下载完整的macOS，就跟白苹果的开机按CMD+R重启到恢复模式一样。Recovery dmg比较小，用一个小u盘就能搞定。

      对于其他版本，也有不同的命令，可以点开[recovery_urls.txt](https://raw.githubusercontent.com/acidanthera/OpenCorePkg/master/Utilities/macrecovery/recovery_urls.txt)查看。

   2. 完整模式

      完整的安装包通常有好几个G，官网下载时间比较长。国内的话可以通过下载其他人，比如黑果小兵，分享的安装包。通过[etcher](https://www.balena.io/etcher/)，将安装包写入我们的u盘，最后在用自己的EFI文件覆盖u盘里的。

2. 写盘

   写盘的关键点在于预留好一块fat32格式的EFI分区，用来存放接下来要制作的EFI文件。方式方法也比较多：

   - windows的磁盘管理器
   - 使用第三方工具，，如[rufus](https://rufus.ie/)、[etcher](https://www.balena.io/etcher/)等，轻松创建USB启动盘
   - 使用命令行工具`diskpart`

### 收集Kexts、Drivers以及SSDT

- SSDTs和DSDTs，以`.aml`为文件名后缀，放到ACPI文件夹下
- Kexts，以`.kext`为后缀，放到Kexts文件夹下
- 固件驱动，以`.efi` 为后缀，放到Drivers文件夹下

TODO

### 配置config.plist

TODO

## 安装调试

TODO

1. bios设置
2. 格式化mac磁盘
3. u盘启动到硬盘启动
4. 2k显示器开启hDPI
5. 自定义USB端口
6. 屏蔽不支持的独显（双显卡以及切换）

可能需要解决的问题：

 - 双显卡及其切换
 - Usb3.0不识别
 - 显示器字体发虚
 - 双系统相关：更改opencore默认启动项；windows正常使用独显
 - 开机黑屏，必须重启显示器或者插拔连接线才能正常工作
 - 蓝牙

## mac Setup

mac Setup是指对一个刚安装好的 mac 系统进行系统配置和软件安装，适用于正规的 macOS 硬件和黑果。针对这部分内容，我会单独再写一篇文章。这里就不再赘述。

## 成果展示环节
- 系统信息  
![系统信息](https://cdn.jsdelivr.net/gh/longf2021/myImage/hackintosh/20210312083021.png)

- 内存  
![内存](https://cdn.jsdelivr.net/gh/longf2021/myImage/hackintosh/20210312083113.png)

- 核显  
![显卡](https://cdn.jsdelivr.net/gh/longf2021/myImage/hackintosh/20210312084248.png)

- 蓝牙和无线  
![蓝牙](https://cdn.jsdelivr.net/gh/longf2021/myImage/hackintosh/20210312083451.png)

- 测试隔空投送功能  
![隔空投送](https://cdn.jsdelivr.net/gh/longf2021/myImage/hackintosh/20210312084101.png)


## 参考资料

1. 《[OpenCore-Install-Guide](https://dortania.github.io/OpenCore-Install-Guide/)》黑果入门级教程，写得非常详细
2. [新手挑战黑苹果-超详细的OpenCore黑苹果安装教程](https://www.bilibili.com/video/BV18V41187JZ) B站上的视频教程

<br> 

<center>  ·End·  </center>
