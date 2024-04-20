---
title: PNG图片数据隐写
date: 2021-03-29 23:34:15
author: repoog
excerpt: 看到推上有人发了一张png图片，保存并修改后缀位zip后竟然是一个压缩包，而且解压后的内容是可以正常运行的Python程序。这引起了我的好奇，于是照葫芦画瓢仿制了一个更为简单的实现（原作者认为考虑的情况不够周全，故拒绝了PR），同时分析了这种现象的原理。
comments: true
tags:
  - PNG
  - Twitter
  - 隐写
  - 数据隐藏
categories:
  - 软件安全
---

Twitter上有人发出一张[png图片](https://twitter.com/David3141593/status/1371974874856587268)，保存后修改后缀为zip，便是一个zip压缩包，解压后是一个Python项目，即tweetable-polyglot-png，可以发推的变形png文件，绕过twitter对图片的处理。图片隐写很常见，比如早年间通过copy命令将文本拷贝在图片里，但这样的png图片隐写，可以保证数据恢复的完整性、易用性。在需要隐蔽的数据传输场景下，是绝佳的传输方式。比如《[Hackers hide credit card data from compromised stores in JPG file](https://www.bleepingcomputer.com/news/security/hackers-hide-credit-card-data-from-compromised-stores-in-jpg-file)》中，攻击者盗取Magento电商系统中的支付卡信息，就是将数据写入了图片文件，再通过图片获取数据，避免数据传输被监测到。

PNG（Portable Network Graphics，便携式网络图形），是一种无损压缩的位图图片格式，最初始开发的目的是为了替代GIF格式的图片，GIF图片因为专利问题而无法使用（专利已在2003年失效）。相比JPEG、GIF等压缩图片，PNG图片支持无损压缩和透明度，但同等复杂度的图片，PNG也会相比JPEG、GIF格式的图片体积大，因此在GIF压缩算法专利失效后，网站中的图片会使用体积相对较小的GIF格式图片，而PNG因为支持真彩色，因此图片清晰度比GIF要好。

PNG是一种位图格式，结构由一个文件署名（Signature）和数据块（Chunk）构成，数据块分关键数据块（IHDR、PLTE、IDAT、IEND）和辅助数据块两种。

文件头是8个字节：

![PNG文件头](images/2021/03/png_header.png 'PNG文件头')

数据块由四个部分组成：

![PNG数据块](images/2021/03/png_data_chunk.png 'PNG数据块')

关键数据块类型如下：

**IHDR（文件头数据块）**

必须作为第一个数据块，用13个字节表示图像数据的基本信息。

![IHDR](images/2021/03/IHDR.png 'IHDR')

**PLTE（调色板）**

包含了索引色图像（indexed color image）的相关数据，如果存在，要放在图像数据块（IDAT）之前。调色板实际上是一个彩色索引表，表的项目数范围是 1~256，每个项有 3 个字节分别用来储存RGB 三个值，所以调色板数据块最大字节数为 768。必须在IHDR数据块之后。

**IDAT**

存储实际的图片数据，一个图片数据流可能包含多个连续顺序的IDAT数据块，数据内容是经过压缩的图片数据。IDAT数据块不是必要的，也可以同时存在多个，但是它们必须是连续排列的。如果存在，必须在IHDR和PLTE数据块之后。

**IEND**

00 00 00 00 49 45 4E 44 AE 42 60 82，表示 PNG 文件结束，必须放在最后面，且 Chunk Data 为空。

zip文件作为压缩文件格式之一，其后缀不是必须的，诸如.JAR、.WAR、.DOCX、.XLSX、.PPTX、.ODT、.ODS、.ODP等后缀文件也是zip文件。zip文件结构由记录、中央目录和中央目录结束三部分构成：

**Local File Header**

![Local File Header](images/2021/03/local_file_header.png 'Local File Header')

**Central Directory Header**

![Central Directory Header](images/2021/03/central_directory_header.png 'Central Directory Header')

**End of Central Directory Record**

![End of Central Directory Record](images/2021/03/end_of_central_directory_record.png 'End of Central Directory Record')

因为IDAT不是必须，且是连续的数据块，因此在做PNG文件的隐写时，可以把隐写的数据通过IDAT写入。以文章开头将zip压缩包写入png图片并绕过twitter系统图片处理为例：

* 打开原始png图片和待写入文件；
* 遍历数据块，获取IHDR数据块中图片的宽、高像素；
* 将IDAT数据块加上待写入文件作为已IDAT数据块，重新计算数据块长度和CRC，写入到输出文件；
* 如果写入文件是zip文件，需要重新计算合并后IDAT数据块中，zip文件中央目录和中央目录结束符相对整个IDAT数据块的文件偏移；
* 输出文件写入IEND数据块；

图片上传twitter后，为了节省存储大小，会被twitter做图片二次压缩或格式转换，只有长/宽小于900像素的png图片可以保证不被做格式转换：

![压缩的png图片](images/2021/03/twitter_image_support.png '压缩的png图片')

如果只是简单的将zip或rar文件置于png文件中，实现修改后缀即可解压的效果，同样使用Windows下的copy命令即可。

> copy /b for\_you.png + hide\_png.rar call\_me.png

![待处理图片](images/2021/03/call_me.png '待处理图片')

如果将上面的图片上传twitter，经过处理后的图片虽然格式不变，但会被清除掉IEND数据块后面附加的文件，导致隐藏文件丢失。

![图片格式对比](images/2021/03/origin_vs_twitter.png '图片格式对比')

如果将IEND数据块换到附加的zip文件后，上传图片时会提示“你上传的一些图片失败”。所以符合png文件格式是必须的，但实际上，并不需要处理zip文件的中央目录和中央目录结束的偏移，只需要将隐藏的数据附加到png中IDAT数据块并计算CRC32即可。