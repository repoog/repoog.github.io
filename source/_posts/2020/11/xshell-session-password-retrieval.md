---
title: XShell的session密码找回
date: 2020-11-02 21:51:01
author: repoog
excerpt: XShell是很好用的终端管理工具，但如果忘记了会话的密码是一件很麻烦的事，本文介绍的是如果用简单的Python代码找回被遗忘的会话密码，或者叫做服务器连接密码。
comments: true
tags:
  - session
  - Xdecrypt
  - XShell
  - 密码
  - 密码找回
categories:
  - 系统安全
---

XShell是很好用的终端管理工具，尴尬的是如果用帐号/密码方式登录服务器并保存XShell会话（XShell session），有一天可能会忘记服务器密码，要么继续使用已保存的会话，要么重置服务器密码重新创建/编辑会话。XShell会话中保存的服务器密码是哈希存储在会话文件中的，但并非无法找不回。

XShell的会话目录可以通过“工具——选项——常规——会话”查看，会话文件是.xsh后缀，实际是INI配置文件，保存了服务器的IP、端口、用户名以及哈希后的密码。

XShell不同版本使用的会话密码哈希方式和算法略有不同，分为5.1之前版本、5.1版本和5.1之后版本。我用的是5.1之后版本的XShell，根据[《How does Xmanager encrypt password?》](https://github.com/HyperSine/how-does-Xmanager-encrypt-password/blob/master/doc/how-does-Xmanager-encrypt-password.md)可知，该版本的密码哈希流程如下：

![password generation of xshell](images/2020/11/xsh_pass.jpg 'XShell会话密码存储生成过程')

[https://github.com/dzxs/Xdecrypt](https://github.com/dzxs/Xdecrypt)便是基于上面的密码加密过程进行解密的，使用前时需要安装pywin32库获取当前系统账户名和SID，该项目可以凝练为以下代码：

``` Python
ARC4.new(SHA256.new(USER+SID.encode('ascii')).digest()).decrypt(base64.b64decode(PASSWORD)[:len(base64.b64decode(PASSWORD)) - 0x20]).decode('ascii')
```

其中USER和SID分别是系统账户名和SID，PASSWORD指.xsh文件中的PASSWORD值。效果如下：

![xshell password decrypt](images/2020/11/xsh_plain.png 'XShell会话存储密码还原')