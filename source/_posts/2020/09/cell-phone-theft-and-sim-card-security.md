---
title: 手机失窃和SIM卡安全
date: 2020-09-27 00:12:23
author: repoog
summary: 前段时间看到一篇文章，是关于手机失窃后自己的资金被黑色产业链盗走的，虽然原文已经被删除，但可以反应除移动安全或者手机安全方面普通人应当注意到的安全风险。本文进一步探究了SIM卡安全的相关问题，并亲身实践发现结合社工手段，即便设置SIM卡密码也依然存在巨大的风险。
comments: true
tags:
  - 2FA
  - OTP
  - SIM卡
  - 手机失窃
  - 手机安全
categories:
  - 移动安全
---

前段时间，一篇《一部手机失窃而揭露的窃取个人信息实现资金盗取的黑色产业链》被很多人转载，SIM卡安全再次被重视起来。原文已被作者删除，根据其他人的总结，坏蛋们的套路如下：

1. 在营业厅下班后偷手机换SIM卡到其他手机；
2. 使用短信验证码登录运营商官网，更换服务密码；
3. 使用短信验证码登录社保网站，获取身份证号、银行卡号、证件照片；
4. 使用短信验证码登录风控薄弱的金融APP贷款、套现；

看完文章，吓得我赶紧查看了自己手机SIM卡的PIN（Personal identification number）码是否设置，记得自己设置过PIN码，却不巧三次都输错，导致SIM卡被锁，锁定的SIM卡需要输入PUK（Personal Unblocking Key）码，却又不记得运营商分配的PUK码，PUK码连续输错10次SIM卡会被永久锁定，只能到营业厅补卡。手机因此无法使用，如果设置PIN码，重启手机或更换设备都需要先输入PIN码。

在PUK码输入界面，手机只能紧急呼叫，不能拨打运营商电话，因此只能拔卡连Wifi使用。PUK码忘记可以在网上营业厅的个人资料中查看，或者咨询线上人工客服（需要登录），偏偏服务密码又忘记，找回服务密码又需要短信验证码或个人身份信息（我用北京联通，23点-7点无法使用该方法），直到第二天使用身份证照片才找回服务密码。通过线上人工客服，仅需要提供身份证号码和姓名，就可以找回PUK码，从而进一步重设PIN码。

整个环节的薄弱点包括：个人真实姓名、身份证号码、身份证照片、服务密码、PIN码、短信验证码。因此仅仅设置PIN码对于稍微用心点的大坏蛋来说是不够的，还包括保护个人身份信息和服务密码。现在大多数主流应用还在使用短信验证码作为2FA或者OTP的方式，而短信本身就面临不止SIM卡被窃这一种风险，还包括SIM Swap、伪基站、短信窃听等等问题。因此，NIST（美国国家技术标准研究所）在2020年发布的《SP 800-63 Digital Identity Guidelines》中将SMS列为RESTRICTED authenticator，即不可靠的认证方式：

> Currently, authenticators leveraging the public switched telephone network, including phone- and Short Message Service (SMS)-based one-time passwords (OTPs) are restricted. Other authenticator types may be added as additional threats emerge. Note that, among [other requirements](https://pages.nist.gov/800-63-3/sp800-63b.html#pstnOOB), even when using phone- and SMS-based OTPs, the agency also has to verify that the OTP is being directed to a phone and not an IP address, such as with VoIP, as these accounts are not typically protected with multi-factor authentication.
> 
> [https://pages.nist.gov/800-63-FAQ/](https://pages.nist.gov/800-63-FAQ/)

另外，虽然国内运营商的要求，让SIM Swap的攻击不会发生，补卡只能老老实实去营业厅办理。但手机号被运营商回收再被其他人领取其实就是间接的SIM Swap，有厂商早些年称之为“手机号中毒”，所以换手机号要谨慎，换号前要做好各个应用换号的准备。我曾经用的一个号码，老是被陌生人打，叫的都是同一个人名字，好奇心驱使，用这个号码尝试登录某社交APP，之前的用户长相却有三分帅，而假如我用号码做借贷，他就会莫名得到三分利了。