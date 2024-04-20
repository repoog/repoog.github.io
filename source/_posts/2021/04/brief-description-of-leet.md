---
title: 简述Leet（1337）
date: 2021-04-27 13:42:27
author: repoog
excerpt: 本文简要介绍Leet 1337的编码方法，该方法被广泛运用于早期各个技术论坛或BBS，之于中文相当于火星文，是国外黑客文化的一种体现。这种编码方式无关乎真正意义上的字符编码，而是一种文化之下的字符替换思路，通常是人工替换完成的。
comments: true
tags:
  - 1337
  - Leet
  - 口令安全
  - 黑客代码
categories:
  - 黑客文化
---

Leet是英文elite（精简）衍生的叫法，也可以叫leetspeak，或者黑客语，是上世纪80年代，极客和黑客门内在BBS中常使用的一种文字书写方式，类似中文世界中的火星文或拆体字。当时用户使用这种表达方式有三种原因：

出于节省流量的目的，比如you改为u，are改为r，类似早期电报和BB机时代通讯时简写的做法。

出于内容检测的绕过，比如porn改为p03n，p()rn。

出于隐秘传输的需要，精简的变形书写能够一定程度上避免圈外人知晓其中的信息，比如：

> You don't know what i'm saying

使用Leet表示，可以是

> j00 )0\\'7 <\\0\\/ \\/-@ 1'\\/ 54\`/1\\'

通常是把字母、数字和特殊符号，通过类似或相同形象的替换、组合，表述同一种意思，且没有统一的标准。比如Leet的另一种文字表达是1337，又或者“you don't know leet”，可以写成“U d0n7 kn0w 1337”。

有很多Leetspeak转换的网站，比如[https://1337.me/](https://1337.me/)、[https://www.dcode.fr/leet-speak-1337](https://www.dcode.fr/leet-speak-1337)，Google也有使用Leet文字的搜索方式：[https://www.google.com/?hl=xx-hacker](https://www.google.com/?hl=xx-hacker)

用在口令设置中，Leet可以一定程度上避免口令被猜解，因为不同的人有不同的字符替换或组合习惯，相同的明文口令，经过替换之后，可以变为强度更高的口令，比如：

> ThisIsMyPassword

通过Leet转换后，会变成：

> 7hIsisMYpA22W0RD

又或者

> 7H1$1$\\/\`/>4$$\\/0.-)

这样原本纯字母的口令，会转换为包换大小写字母、数字和特殊符号的口令，即便被攻击者猜测到口令，也无法猜测到习惯，从而增加口令被猜解的难度，同时方便口令记忆。