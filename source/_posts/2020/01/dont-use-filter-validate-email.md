---
title: 别用FILTER_VALIDATE_EMAIL
date: 2020-01-13 17:08:38
author: repoog
excerpt: 本文介绍的是PHP自带的过滤函数filter_var对于邮箱地址有效性检测的不足，通过构造特殊的邮箱地址，可以绕过该过滤函数的邮箱检测机制，从而触发XSS漏洞。
comments: true
tags:
  - FILTER_VALIDATE_EMAIL
  - filter_var
  - PHP
  - 邮箱安全
  - XSS
categories:
  - 软件开发
---

PHP的过滤器函数filter\_var可以通过FILTER\_VALIDATE\_EMAIL选项判断字符串是否是邮箱格式，如果是则返回过滤后的字符串，不是则返回FALSE。

但这个邮箱过滤的方式极其地粗糙，使用不当很容易造成安全问题。

FILTER\_VALIDATE\_EMAIL的实现在 [https://github.com/php/php-src](https://github.com/php/php-src) 的ext/filter/logical\_filters.c文件中可以找到，对应的是该文件的 void php\_filter\_validate\_email(PHP\_INPUT\_FILTER\_PARAM\_DECL) 函数。

从具体实现可以知道，FILTER\_VALIDATE\_EMAIL本质上还是基于正则表达式实现，用于判断的正则表达式有两种，取决于filter\_var函数调用时是否使用FILTER\_FLAG\_EMAIL\_UNICODE标志。不用该标志的情况下，使用的是以下正则表达式：

``` C
const char regexp1[] = "/^(?!(?:(?:\\x22?\\x5C[\\x00-\\x7E]\\x22?)(?:\\x22?[^\\x5C\\x22]\\x22?)){255,})(?!(?:(?:\\x22?\\x5C[\\x00-\\x7E]\\x22?)(?:\\x22?[^\\x5C\\x22]\\x22?)){65,}@)(?:(?:[\\x21\\x23-\\x27\\x2A\\x2B\\x2D\\x2F-\\x39\\x3D\\x3F\\x5E-\\x7E]+)(?:\\x22(?:[\\x01-\\x08\\x0B\\x0C\\x0E-\\x1F\\x21\\x23-\\x5B\\x5D-\\x7F](?:\\x5C[\\x00-\\x7F]))*\\x22))(?:\\.(?:(?:[\\x21\\x23-\\x27\\x2A\\x2B\\x2D\\x2F-\\x39\\x3D\\x3F\\x5E-\\x7E]+)(?:\\x22(?:[\\x01-\\x08\\x0B\\x0C\\x0E-\\x1F\\x21\\x23-\\x5B\\x5D-\\x7F](?:\\x5C[\\x00-\\x7F]))*\\x22)))*@(?:(?:(?!.*[^.]{64,})(?:(?:(?:xn--)?[a-z0-9]+(?:-+[a-z0-9]+)*\\.){1,126}){1,}(?:(?:[a-z][a-z0-9]*)(?:(?:xn--)[a-z0-9]+))(?:-+[a-z0-9]+)*)(?:\\[(?:(?:IPv6:(?:(?:[a-f0-9]{1,4}(?::[a-f0-9]{1,4}){7})(?:(?!(?:.*[a-f0-9][:\\]]){7,})(?:[a-f0-9]{1,4}(?::[a-f0-9]{1,4}){0,5})?::(?:[a-f0-9]{1,4}(?::[a-f0-9]{1,4}){0,5})?)))(?:(?:IPv6:(?:(?:[a-f0-9]{1,4}(?::[a-f0-9]{1,4}){5}:)(?:(?!(?:.*[a-f0-9]:){5,})(?:[a-f0-9]{1,4}(?::[a-f0-9]{1,4}){0,3})?::(?:[a-f0-9]{1,4}(?::[a-f0-9]{1,4}){0,3}:)?)))?(?:(?:25[0-5])(?:2[0-4][0-9])(?:1[0-9]{2})(?:[1-9]?[0-9]))(?:\\.(?:(?:25[0-5])(?:2[0-4][0-9])(?:1[0-9]{2})(?:[1-9]?[0-9]))){3}))\\]))$/iD";
```

稍微整理下再看：

```
/^(?!(?:(?:"?\[-~]"?)  (?:"?[^\"]"?)){255,})
(?!(?:(?:"?\[-~]"?)  (?:"?[^\"]"?)){65,}@)
(?:(?:[!#-'*+-/-9=?^-~]+)  (?:"(?:[--!#-[]-]  (?:\[-]))*"))
(?:\.(?:(?:[!#-'*+-/-9=?^-~]+)  (?:"(?:[--!#-[]-](?:\[-]))*")))*
@
(?:(?:(?!.*[^.]{64,}) (?:(?:(?:xn--)?[a-z0-9]+(?:-+[a-z0-9]+)*\.){1,126}){1,}
(?:(?:[a-z][a-z0-9]*)  (?:(?:xn--)[a-z0-9]+))(?:-+[a-z0-9]+)*)  (?:\[(?:(?:IPv6:(?:(?:[a-f0-9]{1,4}(?::[a-f0-9]{1,4}){7})  
(?:(?!(?:.*[a-f0-9][:\]]){7,})(?:[a-f0-9]{1,4}(?::[a-f0-9]{1,4}){0,5})?::(?:[a-f0-9]{1,4}(?::[a-f0-9]{1,4}){0,5})?)))  
(?:(?:IPv6:(?:(?:[a-f0-9]{1,4}(?::[a-f0-9]{1,4}){5}:)  (?:(?!(?:.*[a-f0-9]:){5,})(?:[a-f0-9]{1,4}(?::[a-f0-9]{1,4}){0,3})?::(?:[a-f0-9]{1,4}(?::[a-f0-9]{1,4}){0,3}:)?)))
?(?:(?:25[0-5])(?:2[0-4][0-9])(?:1[0-9]{2})(?:[1-9]?[0-9]))(?:\.(?:(?:25[0-5])(?:2[0-4][0-9])(?:1[0-9]{2})(?:[1-9]?[0-9]))){3}))\]))$/iD
```

其中一些十六进制转换成非打印字符而显示成□，不过不要紧。重点是可以看出，这个正则是可以匹配诸多特殊字符的，比如!#-\'\*+-/-9=?^-\~\"，甚至支持punycode，差不多只相当于判断了@ 符号的存在。通过[RFC822 email address validator](http://sphinx.mythic-beasts.com/~pdw/cgi-bin/emailvalidate)这个在线邮箱地址有效性检测工具，可以看到下面的邮箱都会被认为是有效的。

```
!#-'\*+-/-9=?^-~@126.com
```

```
\"\<script\>\"@126.com
```

```
script@xn--6qq308a
```

因此，在brutelogic的XSS邮箱测试（ [https://brutelogic.com.br/tests/input-formats.php?email=user@domain.com](https://brutelogic.com.br/tests/input-formats.php?email=user@domain.com) ）中，可以通过设置email参数

``` HTML
\"<svg/onload=alert(1)\"@x\.y
```

来触发XSS。