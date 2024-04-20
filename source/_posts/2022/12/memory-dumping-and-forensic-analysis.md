---
title: 内存转储及取证分析
date: 2022-12-05 13:45:54
author: repoog
excerpt: 内存取证是电子取证的一部分，也是电子取证中相对较难的部分。CTF比赛中常常有关于内存转储或取证的题目，但大多数题目在设计上相对粗浅，或不贴近实际工作。本文介绍的是基于Volatility工具的内存转储和取证分析的基本原理和操作。
comments: true
tags:
  - LiME
  - Volatility
  - 内存分析
  - 内存取证
  - 内存转储
categories:
  - 系统安全
---

在进行内存取证分析之前，需要先获得被取证系统的内存转储文件，而后再使用Volatility工具对转储文件进行取证分析。

Volatility是一款开源的内存取证工具，用于应急响应和恶意软件分析 ，作者是Aaron Walters，一个专业内存取证专家，它使用Python开发，且支持Windows（32位/64位）、Mac OS X以及Linux系统。

2019年，Volatility基金会发布了对该框架的全面重写，即[Volatility 3](https://github.com/volatilityfoundation/volatility3) ，相比Volatility 2，版本3的运行速度更快，命令也更加简洁，并支持多种内存格式文件，如.raw、.mem、.dmp等。

### **Windows内存转储**

Windows系统可以使用DumpIt或FTK imager获取内存转储文件。DumpIt直接执行即可获得当前系统的内存转储文件，文件默认保存在当前路径下，而FTK imager需要安装应用程序，如果直接在被取证系统安装，会污染系统硬盘。

从Windows 10和Windows Server 2016开始，微软增加了基于虚拟化的安全（VBS）的功能。 该功能套件包括凭证防护、设备防护、应用程序防护等，这些功能都利用了虚拟安全模式（VSM），VSM包括安全内核和安全用户模式组件。VBS会影响到内存取证或内存转储的操作，内存转储工具通常是将设备驱动加载到内核中，然后通过映射DevicePhysicalMemory对象，使用MmMapIoSpace等函数或直接操纵页表来读取内存。在启用了VSM的系统上使用内存转储工具会导致系统崩溃，即BSOD（Blue Screen Of Death，蓝屏）。

确定VSM是否在实时系统上被启用的方法是检查 "安全系统（Secure System） "进程是否正在运行。或者通过CMD输入msinfo32打开系统信息，在“系统摘要”中有“基于虚拟化的安全性”项目，可以查看该项是否是“正在运行”。

![VSM运行状态](images/2022/12/VMS-Status.png 'VSM运行状态')

网上能够下载的DumpIt工具大多数是1.3.2版本，该版本发布于2011年，不适于已经启用VBS/VSM的Windows 10和Windows 11系统，即便是通过安全中心关闭内存完整性也无法解决BSOD问题（禁用VBS功能可以大幅度提升系统性能，对于游戏玩家或硬件配置较低玩家适用，但会降低系统安全性）。

可以通过https://magnetidealab.com/ 注册下载新版本的DumpIt工具，或点击[这里](https://1drv.ms/u/s!AtipXnOOFZk7igPf8JyRqsQFX8rI?e=ZhuaAf) 下载新版本DumpIt。

### **Linux内存转储**

Linux系统可以使用[LiME](https://github.com/504ensicsLabs/LiME) 获得内存转储文件。安装LiME需要首先安装以下必要的组件和程序：

``` Shell
sudo apt-get install -y linux-headers-$(uname -r) build-essential make gcc lime-forensics-dkms
```

在Kali下，linux-headers的安装可以取消sources.list中source packages的注释，然后更新源后再安装。

接着下载LiME的源码进行编译：

``` Shell
git clone https://github.com/504ensicsLabs/LiME.git
cd LiME/src/
sudo make
```

编译之后会生成lime.ko的文件，该文件即是获得内存转储文件的关键。使用inmod加载该文件至内核，并指定路径和导出的文件格式，路径可以指定为TCP端口，再通过nc重定向到另一台主机，也可以指定本地路径，文件格式默认为lime，方便使用Volatility工具进行内存分析。

``` Shell
insmod lime-6.0.0-kali3-amd64.ko "path=../kali.mem format=lime"
```

命令执行成功后，会导出的内存转储文件kali.mem，同时通过lsmod grep lime可以看到内核文件已加载成功。

卸载lime内核文件可以使用rmmod命令：

``` Shell
rmmod lime
```

### **内存取证**

Volatility3无需再使用--profile参数指定系统类型，可以直接对内存转储文件进行分析，包括进程、网络连接、命令记录等等。要对不同类型的系统进行分析需要Volatility3有对应系统的系统调试符号表（System Debug Symbol）文件，工具自带的符号表文件不一定能够和当前系统匹配。

以Kali系统为例，首先使用uname -r获取系统版本信息，而后使用apt get命令下载对应的符号表文件：

``` Shell
sudo apt install linux-headers-6.0.0-kali3-amd64
```

下载完成后会在/usr/lib/debug/boot/路径下出现vmlinux-6.0.0-kali3-amd64和System.map-6.0.0-kali3-amd64文件。

接着，下载和编译[dwarf2json](https://github.com/volatilityfoundation/dwarf2json) 工具，该工具要求Go 1.14+版本进行编译，下载dwarf2json源码后执行go build进行编译即可。

使用dwarf2json执行以下命令生成符号表文件的json格式文件：

``` Shell
./dwarf2json linux --elf /usr/lib/debug/boot/vmlinux-6.0.0-kali3-amd64 --system-map /usr/lib/debug/boot/System.map-6.0.0-kali3-amd64  xz -c > kali.json.xz
```

最后，将生成的kali.json.xz文件复制到volatility3/volatility3/symbols目录下，执行：

``` Shell
python3 vol.py -f kali.mem linux.pslist
```

如果能够正常输出进程列表，则内存转储文件解析成功。

![kali内存分析](images/2022/12/vol-kali-pslist.png 'kali内存分析')