---
title: Linux声卡无法识别问题的解决
comments: true
tags:
  - Parrot OS
  - Intel声卡
  - 声音输出
  - 设备识别
categories:
  - 系统安全
date: 2024-05-31 22:11:16
author: repoog
excerpt: 对于一些笔记本电脑的内置声卡设备，比如Intel的Tiger Lake系列的声卡，Linux发行版不一定能够正常识别声卡设备，导致系统没有声音输出。本文介绍的是如何快速解决该问题的办法。
---

笔者使用的是Parrot OS，作为一款不怎么流行的Linux发行版，Parrot OS主要是被安全人员所使用，另一个比它更为流行的安全人员用的Linux发行版是Kali。

相比Kali，Parrot OS的社区资料要少很多，而笔者之前一直没有注意到，笔记本的该系统中没有声音输出，在系统的Audio Preference中Output显示的是dummy ouput，硬件设备是HDA NVidia，显然这个设备不是声卡。因此，需要将系统的声卡正确识别并将声音输出改为那个声卡。

首先确定的是，笔记本的声卡设备是正常的（因为是Windows下是可以正常使用的，笔者用的是Dual System），其次，需要在Parrot OS下检查设备和驱动的状况。

通过以下命令，可以检查声卡设备的信息：

``` Shell
sudo lspci | grep audio
```
lspci命令可以显示所有连接PCI总线的设备信息，包括显卡、声卡、网卡等，通常用于硬件诊断、驱动程序问题排查以及了解系统中有哪些PCI设备。
![lspci输出结果](/images/2024/05/lspci.png "lspci输出结果")

从命令输出可以看出声卡设备是Intel Corporation Tiger Lake-H HD Audio Controller，同时也说明硬件连接是正常的。

接着是检查驱动程序是否正常，通过以下命令检查Intel声卡模块是否有正确加载：
``` Shell
lsmod | grep snd
```
该命令的输出显示Intel声卡模块snd_hda_intel是正常的。

最后是检查声音管理服务alsa是否正常，包括alsactl、aplay、alsamixer等命令均可以正常使用，其中alsamixer命令可以更直观看到声卡输入和输出的设备中只有HDA NVidia设备。

检索一番资料后，发现需要在/etc/default/grub内容中的GRUB_CMDLINE_LINUX_DEFAULT改为以下内容：
``` Shell
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash snd_hda_intel.dmic_detect=0"
```
即在"quiet splash"之后再添加"snd_hda_intel.dmic_detect=0"，修改配置之后，最后执行下面命令更新GRUB引导程序，之后重启即可正常识别Intel声卡：
``` Shell
sudo update-grub
```
![aplay声卡识别](/images/2024/05/aplay.png "aplay声卡正常识别")

上述配置中的dmic_detect=0是指在Linux内核加载的时候禁用DMIC（Digital Microphone Interface，数字麦克风接口）检测，DMIC检测的目的是确保系统能够识别和使用数字麦克风，从而提供音频输入功能，但一些声卡设备经过DMIC检测后会与驱动程序产生不兼容的问题，导致系统启动后声卡无法正常工作，因此需要在内核载入时候提前禁用DMIC检测。