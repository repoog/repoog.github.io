---
title: 利用ANSI转义序列在HTTP响应头画图
date: 2020-08-14 18:54:41
author: repoog
excerpt: 看到有人在推上发了一条在HTTP Response Header中呈现的卡通图案，感觉很有趣，但对方用的并非是常用的Apache或者Nginx服务，所以采用类似的思路，用Nginx+PHP实现了类似的效果。这同时也可以看到ANSI转义序列在Web安全中的风险和应用。
comments: true
tags:
  - ANSI ESCAPE SEQUENCE
  - ANSI转义
  - HTTP HEADER
  - Nginx
  - 日志注入
categories:
  - Web安全
  - 奇淫技巧
---

之前无意中看到一条有趣的推，@[thingskatedid](https://twitter.com/thingskatedid) 在HTTP Response Header中画了一只卡通图案：

![HTTP Response Header的卡通图案](images/2020/08/http_moomin.png 'HTTP Response Header的卡通图案')

作者利用的是Varnish缓存服务器的脚本，在HTTP Response Header中使用ANSI Escape Sequence实现的。于是想照猫画虎换一种方式来实现。

ANSI Escape Sequence（ANSI转义序列）是文本终端的一种转义规则，在鼠标出现之前，可以通过转义字符在终端设置文本颜色、背景颜色、光标移动、设置终端模式等功能，是一种古老的转义规则（为了兼容Linux，Windows直到Win 10 v1511版本才支持ANSI转义序列）。

以下介绍的是转义序列的控制序列，控制序列的结构是ESC转义符和\[，转义符代码可以是八进制的\\033、Unicode格式的\\u001b、十六进制的\\x1b或者十进制的27。另外，转义序列区分大小写。除控制序列外，还有系统命令序列，结构是ESC转义符和\]。

控制序列的转义规则如下：

* ESC\[{m};{n}**H**：设置光标位置在m行n列
* ESC\[{v}**A**：设置光标向上移动v行
* ESC\[{v}**B**：设置光标向下移动v行
* ESC\[{v}**C**：设置光标向前（通常是向右）移动v个字符
* ESC\[{v}**D**：设置光标向后（通常是向左）移动v个字符
* ESC\[**2J**：清除显示，恢复光标到默认位置
* ESC\[**K**：清除光标位置到行末尾的字符
* ESC\[{30-37}**m**：设置8位文本颜色，30-37分别代表黑、红、率、黄、蓝、紫红、蓝绿、白
* ESC\[{40-47}**m**：设置8位文本背景颜色，40-47颜色顺序和30-37一样
* ESC\[{30-37}**;1m**：设置16位文本颜色，30-37表示颜色同上
* ESC\[{40-47}**;1m**：设置16位文本背景颜色，40-47颜色顺序和30-37一样
* ESC\[**38;5;**{id}**m**：设置256位文本颜色，id范围是0-256
* ESC\[**48;5;**{id}**m**：设置256位文本背景颜色，id范围是0-256
* ESC\[**38;2;**{r};{g};{b}**m**：设置RGB文本颜色
* ESC\[**48;2;**{r};{g};{b}**m**：设置RGB文本背景颜色
* ESC\[**0m**：清除文本样式和颜色设置

使用Nginx配置文件的add\_header指令，并不能另ANSI转义序列有效，只会原样返回字符。于是转而使用PHP的header函数，curl在终端请求，结果有效。最后的问题是如何将图片转成ANSI转义序列文件，找到一个网站可以做转换：[https://dom.hastin.gs/files/image-to-ansi/#copy](https://dom.hastin.gs/files/image-to-ansi/#copy)

终端中执行：

``` Shell
curl -I https://peirs.net/ansi_escape
```

最终效果如下：

![HTTP Ansi Picture](images/2020/08/http_ansi.png 'HTTP Ansi Picture')

如果终端能够显示ANSI转义序列，那么也可以通过恶意植入构造的日志，在后端运维或开发打印日志的时候执行这些转义。十年前就有人批量发掘过这样的漏洞，比如Nginx 0.7.64的转义序列注入漏洞CVE-2009-4487，攻击者通过GET请求向日志中注入转义序列，用的是系统命令序列，POC如下：

``` Shell
curl -kis http://www.example.com/%1b%5d%32%3b%6f%77%6e%65%64%07%0a

echo -en "GET /\x1b]2;owned?\x07\x0a\x0d\x0a\x0d" > payload
nc localhost 80 < payload
```

以上命令中系统命令序列的ESC\]2;{字符}用来设置终端标题，POC中用来植入命令到Web日志。其他转义序列的漏洞原理与此类似。

### 补充（2020年8月）

由于cat命令不会显示非打印字符，直接使用cat命令打印文件时不会看到恶意的转义序列，比如：

![Security cat command](images/2020/12/image.png 'Security cat command')

安全的做法是使用cat -v或less。