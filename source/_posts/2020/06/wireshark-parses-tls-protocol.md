---
title: Wireshark解析TLS协议
date: 2020-06-24 16:35:59
author: repoog
excerpt: 在一些场景下需要对于SSL或TLS加密的通讯进行解密，尤其是遇到二进制应用程序的时候，因此需要能够使用正确的方法对于SSL/TLS通讯进行解密，而最合适的工具莫过于Wireshark。
comments: true
tags:
  - HTTPS
  - SSL
  - TLS
  - Wireshark
categories:
  - 网络通讯
---

前段时间有个大厂的朋友问怎么从Windows客户端软件抓取HTTPS协议的通讯，解决这个问题的方法有很多。使用Wireshark也可以做TLS协议的解析：一种是直接根据RSA私钥进行解密，RSA私钥通常情况下很难获得（没有使用DH密钥交换；协议是SSLv3，TLS 1.0-1.2；必须是服务器证书的私钥；建立连接时必须有ClientKeyExchange握手信息）； 一种是使用会话密钥的日志文件，这是通用办法，即便使用DH密钥交换也可以做解密。

密钥日志文件是系统配置环境变量SSLKEYLOGFILE后浏览器生成的文本文件，也就是每次会话密钥会写入这个文本文件中。Wireshark配置该文件后即可解析抓取的TLS通讯数据。

Wireshark的首选项→Protocols下选择TLS，在其中的(Pre)-Master-Secret log filename中配置环境变量SSLKEYLOGFILE中配置的日志文件路径即可，配置完成后需重启浏览器。TLS debug file可以记录数据解密过程。

![配置Pre-Master-Secret log filename](images/2020/06/image-1.png '配置Pre-Master-Secret log filename')

为了使密钥日志文件解密协议内容生效，还需要配置Protocols下的TCP协议，启用“Reassemble out-of-order segments”，该选项在3.0版本后默认是禁用的。

![配置Reassemble out-of-order segments](images/2020/06/image.png '配置Reassemble out-of-order segments')

配置RSA密钥解密，可以在Protocols下的“RSA密钥”对话框中配置密钥文件，文件格式可以是PEM或者PKCS#12格式（.pfx或.p12后缀）。

Wireshark官网提供了批处理脚本来避免设置全局环境变量SSLKEYLOGFILE，替换firefox为其他应用，也可以解决非浏览器应用的TLS流量解密问题，如果其使用的TLS库支持的话（SSLKEYLOGFILE实际是TLS库生成的）：

``` Shell
@echo off
set SSLKEYLOGFILE=%USERPROFILE%\Desktop\keylogfile.txt
open firefox
```

启用和配置SSLKEYLOGFILE后，就可以通过Wireshark抓包获取TLS的解密数据。

![TLS数据的解密](images/2020/06/image-2.png 'TLS数据的解密')

在了解更多信息后，解决了文章开头朋友的问题，没用以上办法，而是做的逆向，回报是一个大红包。