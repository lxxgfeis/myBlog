---
title: "自己动手安装黑苹果"
date: 2020-12-29T09:25:49+08:00
draft: false
tags: ["macOS"]
categories: ["Mac/Linux杂七杂八"]
featured_image: ""
description: ""
---
## 概述

组黑苹果有两个层次，如下所述：

1. ✅ 能用的黑果

   正确配置 OC 引导，能够正常的进入 macOS，能够安装并使用工作常用的软件。本人系程序猿一枚，常用的工作环境以及软件有 homebrew、golang、docker、vscode 等，这些软件因人而异。

2. ✅ 好用的黑果

   支持蓝牙、随航、隔空投送、正常休眠唤醒、各种 usb 设备正常工作。但这些都是使用黑果的非必要条件。

对于我个人来说，折腾黑果的最大动力就是经济效益。全套下来 `主机（5000RMB）+4K显示器（2000RMB）= 7k` 左右，能够体验到 `20k` 的效果，个人感觉还是超值的。

其次，黑果能够提供给我一个**稳定有效**的工作环境。我不太介意每次开机需要重新插拔一下键盘或者鼠标的 USB 接口，也不太介意能否正常休眠唤醒（PS. 因为是台式机，所以不太需要休眠，能锁屏就行）。所以对我来说，第一目标就可以满足我的需求，第二目标中的一些功能可以等到需要时再慢慢想办法支持。

## 介绍

我最开始接触并动手组建自己的黑果是在 2020 年，这是一个尴尬的时间节点。因为在 2020 年，苹果公司推出了基于 arm 架构的 m1 芯片（据坊间测评，性能提升还蛮多）。这意味着基于x86的黑果可能没有多少日子了。但是，如果是为了工作或者单纯体验一下苹果系统，这并不是很大的问题。

目前，我已成功在两台不同配置的台式机上装上了macOS，分别是：

1. 8700k + 华硕ROG的m10f（z370芯片组）
2. 10700 + 微星迫击炮b460m

其中配置1是2018年组装的，并非以装黑果为目的，安装的时候遇到了一些小问题；而配置2是根据黑果的要求进行的硬件选购，安装起来要方便很多。

对于不同配置，安装过程大同小异，下面就来说说安装黑果的一个大致流程。👇

## 安装的流程步骤

安装大致分为以下5步：

1. 查看（确定）电脑配置
2. 收集（制作）EFI
3. 制作USB启动盘
4. 安装调试
5. 启动后调优

## 配置

自己的配置务必搞清楚！需要搞清楚哪些？可以参考我的配置单。

我将以 8700k 这套为代表来对黑果硬件进行说明。这套攒于2018年，是当时的顶配。目前服役2年多，当时主要是为了玩游戏，并未考虑后面会安装黑苹果。

先列下配置，再说下这套配置的问题，最后在说下配置的选择。

### 配置单

| 配件        | 型号                                                         |
| ----------- | ------------------------------------------------------------ |
| CPU         | Intel(R) Core(TM) i7-8700K CPU @ 3.70GHz                     |
| GPU（显卡） | NVIDIA GeForce GTX 1080 Ti<br />intel UHD630                 |
| 主板芯片组  | Z370                                                         |
| 声卡        | NVIDIA GP102 HDMI/DP @ NVIDIA GP102 - High Definition Audio Controller<br />Realtek ALC1220 @ Intel Kaby Point PCH - High Definition Audio Controller (Audio, Voice, Speech) |
| 网卡        | Intel Ethernet Connection (2) I219-V<br />Realtek RTL8822BE Wireless LAN 802.11ac PCI-E Network Adapter |
| 硬盘        | Samsung SSD 960 EVO 250GB<br />WDC WD20EZRZ-00Z5HB0          |

### 配置问题

这套配置中NVIDIA显卡和Realtek无线网卡是没法驱动的。

对于显卡而言，如果有剪辑视频需求，那么需要另行购入免驱A卡，N卡都是不能驱动的（只能安装较低版本的macOS）。本人主要是用来作后端开发，直接使用8700k自带uhd630核显就可以。

对于无线网卡，本人另外购买了博通的BCM94360CD，这是免驱卡，可以同时解决无线和蓝牙的问题。

### 配置怎么选

网上有很多类似的说明，下面是我的简要心得：

1. cpu选intel的，最好是带核显，amd虽然也行，但是安装起来比较折腾
2. 显卡，最好购买a卡，买之前先查查能否在macOS中免驱，N卡就别想了
3. 无线网卡，买博通的免驱网卡，在windows上也可以用
4. 如果是从零组机，那就照着别人的配置单抄吧。那么，整个安装过程就简单百倍。

其他的好像也没啥特别注意的地方了。如果想详细了解硬件限制，推荐阅读下面这篇文章：[Hardware Limitations](https://dortania.github.io/OpenCore-Install-Guide/macos-limits.html)。

## 制作EFI

最简单的方法就是收集别人做好的EFI。可以到github上以`CPU 芯片组 EFI`为关键词进行搜索，如`8700k z370 EFI`，也可以去远景论坛看看。

但是，我还是推荐自己动手制作EFI，其实也没那么困难。推荐的阅读材料《[OpenCore-Install-Guide](https://dortania.github.io/OpenCore-Install-Guide/)》。对于手动制作EFI的人来说，大概都会看这个install guide吧，多看多动手。

制作EFI，主要分为三大部分：

1. 制作USB启动盘
2. 收集Kexts、Drivers以及SSDT
3. 配置config.plist

👇是一些我认为的**关键术语**，理解后可以更好的阅读《Guide》：

| 术语        | 描述                                                         |
| ----------- | ------------------------------------------------------------ |
| **ACPI**    | 高级配置与电源接口。是一种规范，用来定义固件抽象接口。详细可以阅读 [维基百科](https://en.wikipedia.org/wiki/Advanced_Configuration_and_Power_Interface) |
| **Kexts**   | **k**ernel **ext**ension（内核拓展），可以理解为macOS的驱动  |
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
      # 🔎 参考 install guide 获取详细说明 
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
- Firmware Drivers（固件驱动），以`.efi` 为后缀，放到Drivers文件夹下

以下是8700k+z370，也就是我这套，需要收集的东西：

#### Firmware Drivers

| 名称                                                         | 说明                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [HfsPlus.efi](https://github.com/acidanthera/OcBinaryData/blob/master/Drivers/HfsPlus.efi) | 能够显示 HFS 卷，HFS（分层文件系统，macOS上的文件系统）      |
| [OpenRuntime.efi](https://github.com/acidanthera/OpenCorePkg/releases) | 用作OpenCore的扩展，以帮助修补boot.efi以获得NVRAM修复和更好的内存管理。 |

#### Kexts

| 名称                                                         | 说明                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [VirtualSMC](https://github.com/acidanthera/VirtualSMC/releases) | 仿真macs上的SMC芯片，如果没有此芯片，macOS将无法启动         |
| [Lilu](https://github.com/acidanthera/Lilu/releases)         | 许多其他Kexts依赖Lilu，如AppleALC, WhateverGreen, VirtualSMC，没有Lilu，这些Kexts无法工作 |
| SMCProcessor.kext                                            | 🌡️ VirtualSMC插件，用于监控CPU温度                            |
| SMCSuperIO.kext                                              | 🍃 VirtualSMC插件，用于监控风扇转速                           |
| [WhateverGreen](https://github.com/acidanthera/WhateverGreen/releases) | 显卡驱动                                                     |
| [AppleALC](https://github.com/acidanthera/AppleALC/releases) | 声卡驱动                                                     |
| [IntelMausi](https://github.com/acidanthera/IntelMausi/releases)<br />[LucyRTL8125Ethernet](https://www.insanelymac.com/forum/files/file/1004-lucyrtl8125ethernet/) | 以太网卡驱动。本人是英特尔I219-V，所以选这个，根据自己情况选择。<br />我的另外一台是realtek的2.5G的网卡，所以选LucyRTL8125Ethernet |
| [USBInjectAll](https://bitbucket.org/RehabMan/os-x-usb-inject-all/downloads/) | 可以暂时使用此kext启用所有端，以便确认哪些端口是真正需要注入的。个人理解：搭配config.plist中的XhciPortLimit暂时使用，长远的方法还是需要指定USB端口映射 |
| [XHCI-unsupported](https://github.com/RehabMan/OS-X-USB-Inject-All) | 注入对Intel xHCI controllers的支持。我们的主板不需要，不过放进去也没关系（PS. 如果搞不清是否需要，就放进去）。 |
| 网卡和蓝牙                                                   | 买的是免驱的，都不需要                                       |
| [NVMeFix](https://github.com/acidanthera/NVMeFix/releases)   | 🔌 修正非苹果NVMe的电源管理和初始化                           |
| [SATA-Unsupported](https://github.com/khronokernel/Legacy-Kexts/blob/master/Injectors/Zip/SATA-unsupported.kext.zip) | 没有SATA，不需要                                             |

#### ACPI

✅ 确定cpu的平台？

桌面cpu，8700k的平台是Coffee Lake

✅ 确定需要哪些SSDT？

参考《Guide》，我们需要如下SSDT

| 名称         | 作用                                                         |
| ------------ | ------------------------------------------------------------ |
| SSDT-PLUG    | 修复电源管理                                                 |
| SSDT-EC/USBX | 1. 解决EC与macOS的AppleACPIEC不兼容问题<br />2. 创建一个虚拟的EC来满足macOS的AppleBusPowerController的需要<br />3. AppleBusPowerController也需要USBX设备提供USB电源属性 |
| SSDT-AWAC    | ⏰ 修复系统时钟                                               |
| SSDT-PMC     | 修复NVRAM，但貌似z370主板不需要                              |

综上所述，我们使用将前三个放到ACPI文件夹下就可以了。PS：下载《Guide》中预编译好的就可以，感兴趣的话可以后面在动手自己编译。

### 配置config.plist

*这节大部分上是在翻译《 [guide](https://dortania.github.io/OpenCore-Install-Guide/config.plist/coffee-lake.html#starting-point)》*，还有一些自己的理解。

配置config.plist是比较麻烦的一步，也是最容易出错的一步。建议去睡一觉，睡醒之后再来操作，操作之前，多看几遍教程。

**《guide》上没有提到的内容，不用管，保持默认就好**

配置好以后，可以使用 [OpenCore Sanity Checker](https://opencore.slowgeek.com/ ) 对我们的config.plist进行检查。

选择好CPU和OC的版本**以后**，将config.plist拖进去。根据它给出的提示进行改正，不确定的地方以《guide》为准。

#### ACPI

##### Add

add下的条目与ACPI文件夹下的内容一一对应，这儿[ProperTree](https://github.com/corpnewt/ProperTree)会帮我们弄好，只需要检查一下即可

#### Booter

##### Quirks

检查并改正需要修改的值即可

| 名称                   | 值   | 修改？ | 说明                                                         |
| ---------------------- | ---- | ------ | ------------------------------------------------------------ |
| AvoidRuntimeDefrag     | YES  |        | 修复运行时服务，例如日期、时间、NVRAM、电源控制等            |
| DevirtualiseMmio       | YES  | *      | 减少内存占用，修复z390内存分配问题                           |
| EnableSafeModeSlide    | YES  |        | 让slide变量能在安全模式下使用（🙈 不懂）                      |
| EnableWriteUnprotector | NO   | *      | 写保护，和RebuildAppleMemoryMap是冲突的，我们关掉这个        |
| ProtectUefiServices    | NO   | *      | 保护UEFI的服务不被firmware覆盖。在z390主板上启用             |
| ProvideCustomSlide     | YES  |        | 用于计算slide变量，只有在debug日志中看到“OCABC: All slides are usable! You can disable ProvideCustomSlide!”，才需要关闭 |
| RebuildAppleMemoryMap  | YES  | *      | 生成与macOS兼容的Memory Map                                  |
| SetupVirtualMap        | YES  |        | 修复SetVirtualAddresses对虚拟地址的调用                      |
| SyncRuntimePermissions | YES  | *      | 修复MAT tables对齐，跟RebuildAppleMemoryMap相关              |

#### DeviceProperties

##### Add

| 名称                       | 说明                                                         |
| -------------------------- | ------------------------------------------------------------ |
| PciRoot(0x0)/Pci(0x2,0x0)  | 帧缓冲相关，用于设置核显属性。*AAPL,ig-platform-id*是macOS用来决定核显驱动如何和系统交互的。*framebuffer-patch-enable*是启用WhateverGreen.kext的patch功能。*framebuffer-stolenmem*用于设置核显显存为19MB，通常这个不需要的，因为通过BIOS可以设置为64MB。<br />当使用独显输出时，不需要设置framebuffer-相关。 |
| PciRoot(0x0)/Pci(0x1b,0x0) | 设置声卡布局。我的板载声卡是Realtek ALC1220，通过查询《[AppleALC Supported Codecs](https://github.com/acidanthera/AppleALC/wiki/Supported-codecs)》知道我们的layout有：1, 2, 3, 5, 7, 11, 13, 15, 16, 21, 27, 28, 29, 34。随便选一个填上就好。安装好系统以后，如果声卡不工作，再一个个试就行 |

#### Kernel

##### Quirks

| 名称                    | 值   | 修改？ | 说明                                                         |
| ----------------------- | ---- | ------ | ------------------------------------------------------------ |
| AppleCpuPmCfgLock       | NO   |        | Ivy Bridge以及更老的cpu上关闭cfg lock的                      |
| AppleXcpmCfgLock        | YES  | *      | Haswell以及更新的cpu上关闭cfg lock的（PS：cfg lock也可以通过BIOS来关） |
| CustomSMBIOSGuid        | NO   |        | 和UpdateSMBIOSMode一起使用，当UpdateSMBIOSMode设置为Custom时，执行GUID修补。会影响其他操作系统的启动，不推荐使用 |
| DisableIOMapper         | YES  | *      | 关闭IO mapper（PS：类似BIOS里面关闭vt-d）                    |
| DisableLinkeditJettison | YES  |        | 在OC启动参数不包含 `keepsyms=1`时，让Lilu等Kexts性能更稳定   |
| DisableRtcChecksum      | NO   |        | 阻止AppleRTC写主校验和，对于一些重启后进入安全模式的用户有用 |
| ExtendBTFeatureFlags    | NO   |        | 对那些非apple或者fenvi网卡的人有用                           |
| LapicKernelPanic        | NO   | *      | 阻止内核崩溃，一般适用于惠普的系统                           |
| LegacyCommpage          | NO   |        | 解决SSSE3依赖 🙈，一般跟奔腾64位cpu相关                       |
| PanicNoKextDump         | YES  | *      | 内核崩溃后保留日志                                           |
| PowerTimeoutKernelPanic | YES  | *      | 修复Catalina中电源状态变化时的系统崩溃                       |
| SetApfsTrimTimeout      | -1   |        | 设置trim超时时间，单位微秒。碰到有问题的SSD时可能需要        |
| XhciPortLimit           | YES  | *      | 开启后将usb端口限制在15个内，原因是UsbInjectAll重新实现了内置的macOS功能，而没有适当的当前调整。 |

#### Misc

##### Debug

| 名称            | 值         | 修改？ | 说明                 |
| --------------- | ---------- | ------ | -------------------- |
| AppleDebug      | NO         | *      | 开启boot.efi日志     |
| ApplePanic      | NO         | *      | 将崩溃日志记录到硬盘 |
| DisableWatchDog | YES        | *      | 关闭UEFI watchdog    |
| DisplayLevel    | 2147483650 |        | 展示更多的debug信息  |
| SerialInit      | NO         |        | 设置连续的输出       |
| SysReport       | NO         |        | 系统报告             |
| Target          | 0          | *      | 日志级别             |

##### Security

| 名称                 | 值       | 修改？ | 说明                                                         |
| -------------------- | -------- | ------ | ------------------------------------------------------------ |
| AllowNvramReset      | YES      | *      | 允许重置NVRAM                                                |
| AllowSetDefault      | YES      | *      | 允许设置默认启动项                                           |
| ApECID               | 0        |        | 用于对个性化的安全启动标识符进行网络划分。目前不可用，保持默认 |
| AuthRestart          | NO       |        | 为FileVault 2启用经过身份验证的重启，因此重启时不需要密码。  |
| BlacklistAppleUpdate | YES      | *      | 用于阻止固件更新，用作额外的保护级别，因为macOS Big Sur不再使用run-efi-updater变量 |
| DmgLoading           | Signed   |        | 确保仅加载签名DMG                                            |
| ExposeSensitiveData  | 6        |        | 显示更多调试信息                                             |
| **Vault**            | Optional | *      | 很重要，填错启动不了。“Optional”是要填的值，大小写敏感。     |
| **ScanPolicy**       | 0        | *      | 很重要，填错启动不了。0允许我们查看所有可用的驱动器          |
| SecureBootModel      | Default  | *      | 启用Apple的安全启动。不需要可以填Disabled                    |

#### NVRAM

##### Add

1. 4D1EDE05-38C7-4A6A-9CC6-4BCCA8B38C14

   用于OC的UI缩放，默认即可。  

   - UIScale：ui缩放
     - 01：标准
     - 02：HiDPI
   - DefaultBackgroundColor
     - 00000000：黑
     - BFBFBF00：浅灰

   

2. 4D1FDA02-38C7-4A6A-9CC6-4BCCA8B30102

   OC的NVRAM GUID，主要与RTCMemoryFixup用户有关 。

   

3. 7C436110-AB2A-4BBB-A880-FE41995C9F82

   系统完整性保护位掩码

   | 启动参数       | 说明                                                         |
   | -------------- | ------------------------------------------------------------ |
   | -v             | 启动详细模式。能给你展现🍎logo下面的启动细节。个人感觉不需要，随意。 |
   | debug=0x100    | 禁用macOS的看门狗，防止内核崩溃时重新启动。                  |
   | keepsyms=1     | 这是debug = 0x100的一项辅助设置，告诉操作系统留下内核崩溃日志。 |
   | alcid=1        | 用于设置AppleALC的layout-id，启动参数优先于前面在DeviceProperties中设置的layout-id。常用作调试。 |
   | agdpmod=pikera | A卡用户需要，我等核显用户不要                                |
   | nvda_drv_vrl=1 | 在Sierra和HighSierra的Maxwell和Pascal卡上启用Nvidia的Web驱动程序。 |
   | -wegnoegpu     | 禁用除集成英特尔iGPU之外的所有其他GPU                        |

   | 名称              | 值       | 说明                                            |
   | ----------------- | -------- | ----------------------------------------------- |
   | csr-active-config | 00000000 | 用于禁用SIP，目前不需要。保持默认。             |
   | run-efi-updater   | No       | 防止macOS安装固件更新程序包、防止破坏启动顺序。 |
   | prev-lang:kbd     | en-US:0  | 类型选String                                    |

#### PlatformInfo

生成三码，比较简单。注意好对应关系就成。

- `Type` 对应 Generic -> SystemProductName.

- `Serial` 对应 Generic -> SystemSerialNumber.

- `Board Serial` 对应 Generic -> MLB.

- `SmUUID` 对应 Generic -> SystemUUID.

## 安装调试

### BIOS设置

需要禁用如下（其中vt-d和cfg lock找不着的话，可以在config.plist里面开启相应的参数）：

- Fast Boot 
- Secure Boot
- **VT-d** 
- CSM
- Intel SGX
- Intel Platform Trust
- **CFG Lock** (MSR 0xE2 write protection)

需要启用如下：

- VT-x
- Above 4G decoding
- Hyper-Threading
- Execute Disable Bit
- EHCI/XHCI Hand-off
- DVMT Pre-Allocated(iGPU Memory): 64MB
- SATA Mode: AHCI

### 安装系统

格式化安装mac的硬盘时，分区方式选择GUID partition，文件系统选APFS。接着就按照页面上的指引，对你的mac进行初始化吧。

---

恭喜（👏🎉），到这儿，我们就完成了第一个目标：能用的黑果。后面就是一些调优的工作了，安装实际需求来。

## 启动后调优

1. u盘启动到硬盘启动？

   挂载u盘EFI分区 

   -> 将EFI分区中的EFI文件夹拷贝到黑果主机上 

   -> 卸载u盘EFI分区 -> 挂载安装黑果硬盘的EFI分区

    -> 将EFI文件夹拷贝到主机的EFI分区中

    -> 卸载主机EFI分区

2. 显示器字体发虚？

   1. 更换4k，效果最好
   2. 参考《[一键开启 macOS HiDPI](https://github.com/xzhih/one-key-hidpi/blob/master/README-zh.md)》，开启HiDPI，个人试过，效果不是很好

3. Usb3.0不识别，或者键盘鼠标需要重新插拔才能用

   参考b站up主司波图的视频，定制USB端口映射

4. 开机黑屏，必须重启显示器或者插拔连接线才能正常工作

   TODO

## Mac Setup

mac Setup是指对一个刚安装好的 mac 系统进行系统配置和软件安装，适用于正规的 macOS 硬件和黑果。针对这部分内容，我会单独再写[一篇文章](https://longfeis.me/2021/my-macos-setup/)。这里就不再赘述。

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

## 附录一：英特尔CPU

发展历史

| 芯片架构 | 备注                                             |
| -------- | ------------------------------------------------ |
| 4004     | 1971年11月15日世界上第一块个人微型处理器4004诞生 |
| 8086     | 1978年6月8日                                     |
| 80286    | 1982年2月2日，英特尔的最后一块16位处理器         |
| 80386    | 1985年，开始进入到了32位时代                     |
| Pentium  | 奔腾时代                                         |
| Core     | 酷睿时代                                         |

🖥️ 台式机微架构名称

| Code Name                                                    | Series                                                       | Release       |
| :----------------------------------------------------------- | :----------------------------------------------------------- | :------------ |
| [Yonah, Conroe and Penryn](https://dortania.github.io/OpenCore-Install-Guide/config.plist/penryn.html) | E8XXX, Q9XXX  | 2006-2009 era |
| [Lynnfield and Clarkdale](https://dortania.github.io/OpenCore-Install-Guide/config.plist/clarkdale.html) | 5XX-8XX                                                      | 2010 era      |
| [Sandy Bridge](https://dortania.github.io/OpenCore-Install-Guide/config.plist/sandy-bridge.html) | 2XXX                                                         | 2011 era      |
| [Ivy Bridge](https://dortania.github.io/OpenCore-Install-Guide/config.plist/ivy-bridge.html) | 3XXX                                                         | 2012 era      |
| [Haswell](https://dortania.github.io/OpenCore-Install-Guide/config.plist/haswell.html) | 4XXX                                                         | 2013-2014 era |
| [Skylake](https://dortania.github.io/OpenCore-Install-Guide/config.plist/skylake.html) | 6XXX                                                         | 2015-2016 era |
| [Kaby Lake](https://dortania.github.io/OpenCore-Install-Guide/config.plist/kaby-lake.html) | 7XXX                                                         | 2017 era      |
| [Coffee Lake](https://dortania.github.io/OpenCore-Install-Guide/config.plist/coffee-lake.html) | 8XXX-9XXX                                                    | 2017-2019 era |
| [Comet Lake](https://dortania.github.io/OpenCore-Install-Guide/config.plist/comet-lake.html) | 10XXX                                                        | 2020 era      |

💻 笔记本微架构名称

| Code Name                                                    | Series     | Release       |
| :----------------------------------------------------------- | :--------- | :------------ |
| [Clarksfield and Arrandale](https://dortania.github.io/OpenCore-Install-Guide/config-laptop.plist/arrandale.html) | 3XX-9XX    | 2010 era      |
| [Sandy Bridge](https://dortania.github.io/OpenCore-Install-Guide/config-laptop.plist/sandy-bridge.html) | 2XXX       | 2011 era      |
| [Ivy Bridge](https://dortania.github.io/OpenCore-Install-Guide/config-laptop.plist/ivy-bridge.html) | 3XXX       | 2012 era      |
| [Haswell](https://dortania.github.io/OpenCore-Install-Guide/config-laptop.plist/haswell.html) | 4XXX       | 2013-2014 era |
| [Broadwell](https://dortania.github.io/OpenCore-Install-Guide/config-laptop.plist/broadwell.html) | 5XXX       | 2014-2015 era |
| [Skylake](https://dortania.github.io/OpenCore-Install-Guide/config-laptop.plist/skylake.html) | 6XXX       | 2015-2016 era |
| [Kaby Lake and Amber Lake](https://dortania.github.io/OpenCore-Install-Guide/config-laptop.plist/kaby-lake.html) | 7XXX       | 2017 era      |
| [Coffee Lake and Whiskey Lake](https://dortania.github.io/OpenCore-Install-Guide/config-laptop.plist/coffee-lake.html) | 8XXX       | 2017-2018 era |
| [Coffee Lake Plus and Comet Lake](https://dortania.github.io/OpenCore-Install-Guide/config-laptop.plist/coffee-lake-plus.html) | 9XXX-10XXX | 2019-2020 era |
| [Ice Lake](https://dortania.github.io/OpenCore-Install-Guide/config-laptop.plist/icelake.html) | 10XXX      | 2019-2020 era |

## 参考资料

1. [OpenCore-Install-Guide](https://dortania.github.io/OpenCore-Install-Guide/)黑果入门级教程，写得非常详细
2. [OpenCore-Post-Install](https://dortania.github.io/OpenCore-Post-Install/) 黑果安装后调优
3. [新手挑战黑苹果-超详细的OpenCore黑苹果安装教程](https://www.bilibili.com/video/BV18V41187JZ) B站上的视频教程
4. [10代平台黑苹果Opencore6.0下的声卡驱动&USB定制](https://www.bilibili.com/video/BV1Aa4y1j7CL)
5. [英特尔微处理器列表](https://zh.wikipedia.org/wiki/%E8%8B%B1%E7%89%B9%E5%B0%94%E5%BE%AE%E5%A4%84%E7%90%86%E5%99%A8%E5%88%97%E8%A1%A8)



<center>  ·End·  </center>
