---
title: Swaks测试邮件伪造
date: 2021-09-05 17:39:10
author: repoog
excerpt: 本文介绍的是通过Swaks工具对于不同企业邮箱的邮件伪造试验，并对比相同伪造方法下不同企业邮箱对于伪造邮件的防御能力。其中还发现QQ邮箱或腾讯企业邮箱存在的一个漏洞，但可惜官方并没有认可该漏洞。
tags:
  - DKIM
  - DMARC
  - SPF
  - 邮件伪造
categories:
  - 邮箱安全
---

基于笔者之前的《邮件安全防护与溯源》一文可知，可以通过telnet命令发送邮件，发送邮件是SMTP服务器（MTA）传输邮件的过程，接收端的SMTP服务器（MTA）会校验邮件来源与邮件头信息，并结合发件域名DNS配置的SPF、DKIM、DMARC策略来决定邮件的处理方式，如果发件域名的邮件安全防护配置不足，则存在伪造邮件的可能。

邮件伪造的形式有多种，本质上是和发件方有关，因为正常的收件服务器都会进行SPF、DKIM、DMARC等检查，所以根据发件邮箱、被伪造邮箱以及发件SMTP服务器，可以分为以下情况：

1. 发件邮箱域名与被伪造邮箱域名不同；

2. 发件邮箱域名与被伪造邮件域名相同；

本文中用到四个邮件服务商：QQ邮箱、Gmail、ProtonMail以及自定义域名邮箱（阿里云邮箱）。为了方便测试，使用Swaks工具进行操作，Swaks是perl语言编写，通过telnet发送SMTP指令发送邮件的工具，是一款测试SMTP服务的工具。

### **发件邮箱域名与被伪造邮箱域名不同**

通过邮件发送平台SMTP2GO发送，伪造admin@qq.com发送邮件：

``` Shell
swaks --to 502846123@qq.com --from peiww@fengsec.com --h-From admin@qq.com --h-Subject "QQ Mail" --body "Test Message" --server smtp.qiye.aliyun.com -au <account> -ap <passwd>
```

其中，--from即SMTP命令中的MAIL FROM，--h-From即SMTP命令中的From，后者是出现在邮件收件人的信息，前者是真正发送邮件的邮箱。

![伪造QQ邮箱邮件](images/2021/09/QQ_Mail.png '伪造QQ邮箱邮件')

![伪造Gmail邮箱邮件](images/2021/09/Gmail.png '伪造Gmail邮箱邮件')

![伪造Protonmail邮箱邮件](images/2021/09/ProtonMail.png '伪造Protonmail邮箱邮件')

上面三张图，均是伪造admin@qq.com的邮箱分别向QQ、Gmail和ProtonMail发送邮件，QQ邮箱在收件箱中，只有邮件代发的提醒，邮件源码没有SPF、DKIM、DMARC信息，应该是通过QQ邮箱的X-QQ-ORGSender来获取原始发件邮箱，从而判断代发的，Gmail和ProtonMail邮箱都被识别为垃圾邮件，并提示邮件风险，邮件头中都提示未通过DMARC策略检查，原因是缺少DKIM检查，两个邮箱的DMARC策略配置都相对严格，而发件域名没有配置DKIM，虽然通过了SPF检查，但依然被识别为垃圾邮件并提示风险。

![Gmail邮件头](images/2021/09/Gmail_source.png 'Gmail邮件头')

![Protonmail邮件头](images/2021/09/ProtonMail_source.png 'Protonmail邮件头')

上文的三类邮箱都没有实现真正的邮件伪造，或有代发标识，或有安全提示，且说明仅通过SPF无法阻止邮件伪造。

### **发件邮箱域名与被伪造邮件域名相同**

伪造QQ邮箱给另一个QQ邮箱发邮件：

``` Shell
swaks --to 502846123@qq.com --from 502846123@qq.com --h-From 372453823@qq.com --h-Subject "QQ Mail" --body "Test Message" --server smtp.qq.com -au <account> -ap <passwd>
```

![伪造QQ邮箱发邮件](images/2021/09/spoof_QQ_to_QQ.png '伪造QQ邮箱发邮件')

如上图，伪造QQ邮箱给另一个QQ邮箱发邮件是无法成功的，会被判定是发送大量垃圾邮件而拒绝发送，但如果设定--h-From参数伪造邮件来源，则不会发生该错误，因此QQ邮件在服务器发出邮件时有刻意检查发件人信息和发件人邮箱是否一致，采取类似做法的邮件服务商还有Outlook，会提示以下错误：

> 554 5.2.252 STOREDRV.Submission.Exception:SendAsDeniedException.MapiExceptionSendAsDenied; Failed to process message due to a permanent exception with message Cannot submit message.

Gmail邮箱会在邮件发出时，通过邮件服务器检查发件人和发件信息，并在收件中显示发件人，从而让伪造的邮箱原形毕露，如下图企图伪造sml78z@gmail.com发件，但邮件内容显示的依然是来自发件邮箱：

![伪造Gmail邮箱发邮件](images/2021/09/spoof_Gmail.png '伪造Gmail邮箱发邮件')

公共的邮件服务基本都会加强邮件安全的配置，以及增强发件和收件的安全检查，但对于使用自建或注册方式的企业域名的邮箱，如果没有配置SPF、DKIM、DMARC，安全性将大大降低，比如伪造任意企业邮箱：

``` Shell
swaks --to 502846123@qq.com --from peiww@fengsec.com --h-From rulaifozu@fengsec.com --ehlo fengsec.com  --h-Subject "企业邮箱邮件伪造" --body  "测试邮件内容" --server smtp.qiye.aliyun.com -au <account> -ap <passwd>
```

![伪造阿里云企业邮箱发邮件](images/2021/09/spoof_enter_mail.png '伪造阿里云企业邮箱发邮件')

![企业邮箱SPF记录](images/2021/09/spf_enter_mail.png '企业邮箱SPF记录')

fengsec.com是基于阿里云企业邮箱的自定义邮箱域名，虽然配置了SPF，但依然无法阻止邮件伪造。如果发送fengsec.com域名之外的其他域名邮箱，QQ邮箱收件服务器会检测到真实发件地址与发件人地址不一致，并提醒邮件是代发。

![伪造特斯拉邮箱](images/2021/09/spoof_enter_mail_of_tesla.png '伪造特斯拉邮箱')

但在会话模式下（将相同主题的来往邮件以会话形式展示），如果发送一封以上的相同伪造邮件，QQ邮箱将不会有任何代发提醒，所以以上邮件如果再发一次，收件内容就很难从内容中判别邮件伪造：

![会话模式伪造邮件](images/2021/09/session_spoof_mail.png '会话模式伪造邮件')

peirs.net是基于腾讯企业邮箱的自定义邮箱域名，无论是否按照帮助中心的推荐配置SPF和DMARC，伪造同域名下的其他邮箱发送邮件都会提示代发：

![腾讯企业邮箱伪造邮件](images/2021/09/spoof_enter_QQ_mail.png '腾讯企业邮箱伪造邮件')

综上，个人邮箱选择安全可靠的邮件服务商非常重要，安全的邮件服务商能够在发件和收件端提供除SPF、DKIM、DMARC之外的其他邮件安全检测机制，以防止邮件伪造。企业邮箱选择同样需要选择安全可靠的企业邮件服务，可以确保即便企业域名没有配置安全措施的情况下，能够有额外的安全防护，但作为邮箱管理人员，配置必要的SPF和DMARC是非常有必要的（DKIM需要配置邮件服务器）。