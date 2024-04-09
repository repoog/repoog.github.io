---
title: 使用Tor隐匿命令行请求
date: 2021-01-29 14:20:27
author: repoog
summary: 在系统终端中使用命令行时候也会遇到需要匿名发送请求的场景，这个时候可以选择Tor网络作为匿名请求的传输节点。本文介绍的是如何使用Tor网络构建终端下的匿名请求。
comments: true
tags:
  - tor
  - tor browser
  - torsocks
  - 匿名
  - 命令行
categories:
  - 网络通讯
---

除了浏览器访问，在某些场景下需要尽可能隐匿请求来源，比如在命令行中下载某个文件或发起某个请求，防止对方追溯到自己的IP地址。在网络中隐匿来源的最佳办法，莫过于通过Tor了。

Tor项目目前主要的产品是Tor Browser，默认下载的软件包是基于Firefox修改的Tor浏览器，用户使用浏览器连接Tor访问不可描述的网站。Tor通过在传输协议中的应用层加密实现洋葱路由，即先和**网桥**建立连接将消息发出，消息一层一层的加密包装成像洋葱一样的数据包，并经由一系列被称作洋葱路由器的**中继节点**发送，每经过一个洋葱路由器会将数据包的最外层解密，直至**出口节点**将最后一层解密，出口节点因而能获得原始消息，将消息发送给目标系统。而Tor浏览器只是方便用户交互的应用，并不是真正的Tor网络，Tor Browser启用时，同时开启的tor服务（默认监听本地9050端口）才是Tor网络的应用。

另外，由于某个众所周知的原因，在国内通过Tor Browser建立Tor网络连接需要在浏览器中配置FQ代理，否则无法建立网桥连接。

![Tor网络原理](images/2021/01/tor_browser.png 'Tor网络原理')

在命令行中同样如此，需要先搭建tor服务，能够成功建立网桥连接，然后通过其他应用使用tor服务的代理进行匿名访问。本文的搭建环境和使用的工具是Ubuntu + tor + obfs4proxy + torsocks。如果服务器部署在自由网络，可以不用obfs4做流量混淆直接建立tor网络连接。

部署步骤如下：

1、sudo apt install tor torsocks obfs4proxy -y

2、配置tor的配置文件/etc/tor/torrc，参考Windows系统Tor Browser应用安装目录下的Browser\\TorBrowser\\Data\\Tor\\torrc，增加如下配置：

> UseBridges 1  
> ClientTransportPlugin obfs4 exec /usr/bin/obfs4proxy  
> Bridge obfs4 89.163.181.170:443 A0D27B876F1DD14A15C223F48BD9CD4A6BC4517E cert=nOm4+38yOIZ+91ux/vMUOZjUv6pocGtPkZ1QUXumE03Y8akJmrdCwXzxvQVqVPLMlwQrXA iat-mode=0  
> Bridge obfs4 50.39.226.171:47368 93BBD8F80D5F5A8A55829A3168278327BABC14D7 cert=e7kfc/GAUTzv6OEu/a9zQnzGQu9dzhs4jZSmKCXYCaOVZUf5vci2KKilPzR6pUKiiO9hNA iat-mode=0  
> Bridge obfs4 79.199.47.29:9002 6BF05636116C654B65C3F546414739D164D857F1 cert=KmcvY9E6kf6P9ve9gZl0dg0s4bPV4Ik8u25wuJM0p9XXwC+cCxvI8/2jQhjL1qDFFqt9VQ iat-mode=0

或者通过[https://bridges.torproject.org/](https://bridges.torproject.org/)获取网桥地址，使用来源不明的网桥地址会增加行踪暴露的可能性。

3、直接执行tor命令检查日志信息中的连接过程是否报错，如果成功，会显示Bootstrapped 100%；

4、torsocks \[command\]

torsocks通过封装其他命令，将网络请求通过Tor网络发出，因此可以实现命令行下匿名请求的目的。效果如下。

![直接通过curl请求获得的IP地址](images/2021/01/curl.png '直接通过curl请求获得的IP地址')

![通过torsocks封装curl获得的IP地址](images/2021/01/torsocks.png '通过torsocks封装curl获得的IP地址')