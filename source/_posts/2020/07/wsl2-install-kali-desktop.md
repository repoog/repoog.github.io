---
title: WSL2中使用Kali桌面
date: 2020-07-03 14:26:58
author: repoog
excerpt: 本文介绍WSL中安装了Kali系统后，如果通过宿主机对Kali系统进行桌面访问，而不必使用终端。WSL一直在更新和发展，因此文章中办法在后续WSL的更新中可能会失效，或者有官方自带的桌面访问方式替代。
comments: true
tags:
  - Kali
  - RDP
  - WSL
  - xfce
categories:
  - 操作系统
---

WSL2相比WSL在性能上提升了很多，而且有独立的Linux内核，取代VM绰绰有余。有人想在WSL2中安装Linux桌面版本，但Windows目前还不支持，在官方文档中有说明WSL2目前还不支持GUI。

> Can I run ALL Linux apps in WSL?
> 
> No! WSL is a tool aimed at enabling users who need them to run Bash and core Linux command-line tools on Windows.
> 
> WSL does not aim to support GUI desktops or applications (e.g. Gnome, KDE, etc.)
> 
> Also, even though you will be able to run many popular server applications (e.g. Redis), we do not recommend WSL for hosting production services – Microsoft offers a variety of solutions for running production Linux workloads in Azure, Hyper-V, and Docker.
> 
> [https://docs.microsoft.com/en-us/windows/wsl/faq](https://docs.microsoft.com/en-us/windows/wsl/faq)

所以只能通过远程的方式连接WSL2上的系统中，以Kali为例，需要在系统中执行以下几步：

1. sudo apt update && apt upgrade -y
2. sudo apt install kali-desktop-xfce -y
3. sudo apt install xrdp -y
4. sudo service xrdp start

而后通过Windows本地的RDP连接Kali，或者使用VNC也可以。

![xfce桌面环境](images/2020/07/Kali_xfce.png 'xfce桌面环境')

### 更新（2020年7月）

Kali推出了Win-Kex功能，可以在WSL2的Kali系统中安装VNC服务，并直接启动桌面，无需再从远程桌面连接。

Win-Kex的功能包括：

* 窗口模式：在特定窗口中启动Kali的桌面；
* 无缝模式：和Windows系统共享桌面，包括应用和菜单
* 声音支持
* 普通用户和root会话支持
* 和Windows共享剪贴板
* 多会话支持：root窗口、普通用户窗口、无缝会话同时支持

在Kali中安装Win-Kex：

``` Shell
sudo apt update && sudo apt install -y kali-win-kex
```

在Kali中设置VNC服务的密码：

``` Shell
kex
```

之后可以通过kex --passwd修改该密码。

启动桌面时，使用kex命令，桌面系统的用户权限和kex命令的执行用户相同，默认启动窗口模式：

* kex --win：窗口模式
* kex --esm：窗口和远程会话模式，通过RDP连接进行桌面，效果和上文一样
* kex --sl：无缝模式，共享和Windows的桌面、菜单和应用

进入窗口模式后，通过F8显示窗口菜单，来选择退出或全屏模式。

启动Kex时如果遇到以下错误提示：

> Please try “kex start” to start the service. If the server fails to start, please try “kex kill” or restart your WSL2 session and try again.

执行vncserver --localhost no启动vnc服务后再执行kex命令。