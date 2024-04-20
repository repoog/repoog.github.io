---
title: SSH的ProxyJump参数
date: 2020-10-30 16:15:23
author: repoog
excerpt: 本文介绍了在某些场景下，如何使用SSH的端口转发功能实现对于无法访问或无法连接服务器的连接操作，其中用到了SSH的参数ProxyJump。
comments: true
tags:
  - SSH
  - ProxyJump
  - 端口转发
categories:
  - 网络通讯
---

在某些场景下，SSH无法直接访问服务器，需要通过其他服务器进行代理访问，比如外网服务器访问只允许内网的服务器。这种场景下，一种常用的方式是做端口转发，使用端口转发建立连接，然后再做访问。如果面临更多服务器，就需要建立多个端口转发连接。这种方式费力不讨好。

SSH本身提供了ProxyJump参数，用来进行代理服务跳转，简写参数是-J。

> **\-J** destination  
> Connect to the target host by first making a ssh connection to the jump host described by destination  
> and then establishing a TCP forwarding to the ultimate destination from there. Multiple jump hops may  
> be specified separated by comma characters. This is a shortcut to specify a ProxyJump configuration di‐  
> rective. Note that configuration directives supplied on the command-line generally apply to the desti‐  
> nation host and not any specified jump hosts. Use ~/.ssh/config to specify configuration for jump  
> hosts.

可以直接使用SSH命令连接，也可以使用config配置文件，且支持多个代理服务。比如：

``` Shell
ssh -J user@proxyserver1,user@proxyserver2 user@targetserver
```

效果如下：

![ssh的ProxyJump效果](images/2020/10/ssh_proxyjump.png 'ssh的ProxyJump效果')

使用命令时需要逐个输入代理服务器的密码。图方便可以使用config配置和ssh密钥文件，只需要在config文件中的目标服务器配置下方添加ProxyJump proxyserver即可。比如：

```
Host ProxyServer
    HostName x.x.x.x
    Port 22
    User ubuntu
    IdentityFile ~/.ssh/id_rsa
Host TargetServer
    HostName x.x.x.x
    Port 22
    User nopriv
    IdentityFile ~/.ssh/id_rsa
    ProxyJump ProxyServer
```