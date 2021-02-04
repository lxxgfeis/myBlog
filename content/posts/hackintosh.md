---
title: "Hackintosh"
date: 2020-12-29T09:25:49+08:00
draft: false
tags: ["macOS"]
categories: ["macOS相关就搁这儿吧"]
featured_image: ""
description: ""
---
# WIP 安装黑果回忆录

## 概述

把黑果安装起来能用并不困难，难在让黑果完美工作。

本文主要记录本人在安装黑果时学习到的知识，遇到的问题以及问题的解决方案，并非从0开始的黑果安装教程。

## 介绍

2020年，随着Apple公司转向使用m1芯片，给黑果的未来蒙上了一层阴影。但是，Apple公司仍然会支持intel芯片的产品，也就是说，乐观估计黑果再挺个3～5年应该是不成问题的。

就是在这尴尬的时间节点上，我入了黑果的坑。

目前，已成功在两台不同配置的台式机上装上了macOS，分别是：

1. 8700k + 华硕ROG的m10f
2. 10700 + 微星迫击炮b460m

其中配置1是2018年组装的，并非以装黑果为目的。

这篇文章是根据安装过程的回忆编写的，在整个过程中，我学习到了不少知识，也遇到了很多困难。

## 安装的流程步骤

安装大致分为以下5步：

1. 查看（确定）电脑配置
2. 收集（制作）EFI
3. 制作USB启动盘
4. 安装调试
5. mac setup

下面以配置1，也就是8700k这套，来进行说明。

## 配置

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

对于显卡而言，如果有剪辑视频需求，那么需要另行购入免驱A卡，N卡都是不能驱动的。本人主要是用来作后端开发，直接使用8700k自带uhd630核显就可以；

对于无线网卡，本人另外购买了博通的BCM94360CD，这是免驱卡，可以同时解决无线和蓝牙的问题。

**配置怎么选**

网上有很多类似的说明，下面是我的简要心得：

1. cpu选intel的，最好是带核显，amd虽然也行，但是安装起来比较折腾
2. 显卡，最好购买a卡，买之前先查查能否在macOS中免驱，N卡就别想了
3. 无线网卡，买博通的免驱网卡，在windows上也可以用
4. 如果是从零组机，那就照着别人的配置单抄吧。那么，整个安装过程就简单百倍。

其他的好像也没啥特别注意的地方了。如果想详细了解，推荐阅读下面这篇文章：[Hardware Limitations](https://dortania.github.io/OpenCore-Install-Guide/macos-limits.html)

## 制作EFI

最简单的方法就是收集别人做好的EFI。可以到github上以`CPU 芯片组 EFI`为关键词进行搜索，也可以去远景论坛看看。

但是，我还是推荐自己动手制作EFI，其实也没那么困难。

推荐的阅读材料《[OpenCore-Install-Guide](https://dortania.github.io/OpenCore-Install-Guide/)》。对于手动制作EFI的人来说，大概都会看这个install guide吧，本人看了两遍。

下面的内容主要是本人的一些笔记，仅供参考。

### 术语

本节记录从0开始过程中遇到的一些新的术语。

**ACPI:**  [The Advanced Configuration and Power Interface](https://en.wikipedia.org/wiki/Advanced_Configuration_and_Power_Interface) ，翻译过来就是：[高级配置和电源接口](https://zh.wikipedia.org/wiki/%E9%AB%98%E7%BA%A7%E9%85%8D%E7%BD%AE%E4%B8%8E%E7%94%B5%E6%BA%90%E6%8E%A5%E5%8F%A3)。

**AML**:  ACPI机器语言(ACPI Machine Language)

**ASL**:  ACPI源语言(ACPI Source Language)

**DSL:**  

**Kexts**: 驱动

**SSDT**：辅助系统描述表Secondary System Description Table

**DSDT**: 区分系统描述表(Differentiated System Description Table)

DSDT可以看作是包含大多数信息的主体，而较小的信息位则由SSDT传递。

**XCPM**：XNU's CPU Power Management   ？？？？？？

**AWAC**：

**RTC**：

**MMIO**： 内存映射输入输出

**IRQ**： 中断请求（interrupt request）

**iGPU**：核显

**dGPU**：独显

### ACPI

### Kexts

### 配置config.plist

### 一些难以理解的地方

本节记录一些阅读过程中不理解的地方。

"true" 300 series motherboards(Z370 is excluded)

## 制作USB启动盘

## 安装调试

问题：

双显卡及其切换

Usb3.0不识别

显示器字体发虚

双系统相关：更改opencore默认启动项；windows正常使用独显

开机黑屏，必须重启显示器或者插拔连接线才能正常工作

蓝牙

## mac Setup

本章主要是记录黑果安装后的的一些


<br>
<center>  ·End·  </center>
