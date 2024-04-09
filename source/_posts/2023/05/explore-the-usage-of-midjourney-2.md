---
title: 探索MidJourney的用法（二）
date: 2023-05-30 10:54:56
summary: MidJourney最近非常火热，通过文本生成的图片极其逼真，笔者继续尝试基于官方的操作指令探索MJ的更多用法和效果。
comments: true
tags:
  - AI作图
  - AI技巧
  - MidJourney
categories:
  - 人工智能
---

本文探索更多关于MidJourney的用法，官网社区的热图往往偏向创意性或创造性，接着上一篇文章，笔者期望探索不同的Prompt和用法使用MidJourney创造更为实际的图片。

### **多照片合并设计**

通过两张照片以及Prompt描述，将两张照片按照描述合成为预期的效果，比如笔者期望将一张Logo图片出现在另一张孩子的衣服上，实际效果并不如意。

> <img url> is a logo printed on cloth of boy in <img url> and logo is located on the center of clothes,photographic,high detailed,3d rendering,avatar,bright light,spaceship background,F2.8 --iw 1.5

![](images/2023/05/photo-1.png)

MidJourney在处理图片时无法完整处理原图，会根据原图的特点生成新的图片，且会受到原图的背景等影响，越是复杂的背景，越是无法完整呈现图片主体的内容。如上图，Logo图片在处理后生成了各种不同的Logo样式，但好在Logo确实按照Prompt中所描述出现在孩子衣服的胸口。

由此产生另一个想法，通过真人照片凭空在衣服上制造新的样式，比如在衣服胸口打一个特定的Logo：

> <img url> is a boy model with white sweater,print a text logo of letter R on it with red color,avatar,3d rendering,high detailed --iw 1.8

![](images/2023/05/photo-2.png)

上图中是在孩子衣服胸口打印一个R的文字Logo，并且将原来的衣服颜色更换为了白色。经过几番重新生成和选择，最终效果良好的图片如下：

![](images/2023/05/photo-3.png)

### **传说中的神兽**

一些文化中的动物是人们想象的，想象中的动物其实并不存在，笔者尝试在不直接使用动物名称的前提下，使用人们对动物描述的原文或者类似文字的Prompt描述来生成动物，看是否能够符合印象中动物的样子，比如龙、美人鱼。

龙在中国古代的描述中是这样的：

> 头如骆驼，角如鹿，眼如兔，耳如牛，颈如蛇，腹如蜃，鳞如鲤鱼，爪如鹰，掌如虎。

将上面的描述整理为Prompt之后生成图片，却相差甚远：

> a animal which Head like a camel, horns like a deer, eyes like a rabbit, ears like a cow, neck like a snake, belly like a mirage, scales like a carp, claws like an eagle, palms like a tiger.3d rendering

![](images/2023/05/photo-4.png)

即便简化描述，生成的图片也依然是更多现实中动物的模样，比如《美人鱼》电影中对美人鱼的描述，一半是人，一半是鱼，上半身是人，下半身是鱼。

> an animal half body is human and the other half body is fish

![](images/2023/05/photo-5.png)

显然，上面的Prompt在图片生成过程中只能根据关键词进行拼接生成，并无法产生我们想象中美人鱼的样子，即便将Prompt做进一步的细化描述：

> beautiful Chinese girl with fish tail floating in the sea,she has long black hair and beautiful face,realistic,3d rendering,bright light

![](images/2023/05/photo-6.png)

无论如何用朴素的语言描述美人鱼的样子，最终的成图都是女孩加鱼，而无法将人和鱼结合起来形成人鱼的样子，即便样子恐怖一些。

> a beautiful Chinese girl with a big fish tail floating in the sea,full body,long black hair,beautiful face,realistic,3d rendering,high detailed,bright light

![](images/2023/05/photo-7.png)

如果通过--no参数设置不允许出现脚和腿，那么成图是这样的：

> A Chinese girl with a large fish tail floating on the sea, full body, long black hair, beautiful face, realistic, 3D rendering, high detail, bright light --s 750 --no feet,legs

![](images/2023/05/photo-8.png)

如果再加上不允许出现鱼头，那么成图如下：

> A Chinese girl with a large fish tail floating in the sea, full body, long black hair, beautiful face, 3D rendering, high detail, bright light --s 750 --no feet,legs,fish head

![](images/2023/05/photo-9.png)

笔者英文能力范围内无论使用怎样的描述，都无法通过文字来实现想象中的龙或者美人鱼。

### **多人照片合影**

上传两张原图，或者将一张原图和Prompt中的人物名字结合，产生一组两人合影的图片。但笔者发现，/imagine命令无论引入多少图片，通通会基于Prompt生成一个人物的图片，即在图片处理过程中，图片和Prompt是分别被处理的，Prompt是处理所有图片的提示或依据，并无法通过Prompt来实现基于原图的多人合影。

比如笔者想通过自己孩子照片实现和Elon Musk的合影：

> <img url> is a little Chinese boy standing with Elon Musk on a farm working on the plants,full body,realistic,photographic,sunshine light,sunrising,F2.8 --iw 1.8 --s 750

![](images/2023/05/photo-10.png)

最终的成图是Prompt对于原图的描述，因此变成了孩子与Elon Musk的合体，并不是最初所期望的效果。或者简化Prompt，降低对于成图的干扰：

> this little boy is talking with Elon Musk

![](images/2023/05/photo-11.png)

结果依然是合体人，并没有两个人对话的图片。

### **真人照片换装**

通过--iw参数设置原图的比重最大，通过Prompt来实现人物的换装，如果效果良好，可以实现一键换装的效果。

> <img url> blue dress --iw 2

![](images/2023/05/photo-12.png)

通过blue dress的Prompt，将原图中孩子的红色运动服换位了蓝色裙子，虽然长相发生了变化，但对于换装的效果还是明显的。且显然MidJourney对于人脸的识别和还原还存在很大的困难。

笔者从中找到一张相对而言最像原图的图片：

![](images/2023/05/photo-13.png)

### **照片局部修改**

局部修改是MidJourney尚未推出的功能，结合原图使用Prompt，笔者尝试对照片进行局部修改，并以孩子万圣节的照片为例：

> big mouth --iw 2

![](images/2023/05/photo-14.png)

采用最简单的Prompt，将原图的比重设置最大，成图效果虽然嘴巴变大了，但会因为背景或面部其他装扮的干扰而形成恐怖的效果。即上文所说，越是纯粹的照片越能够产生期望的效果，越是复杂的图片主体的处理越会受到其他部分的干扰。

于是索性做了一个可爱版本的小怪物：

> <img url> big mouth,long ears,bright eyes,sharp teeth,realistic,3D rendering,avatar,warm light --iw 1 --ar 9:16

![](images/2023/05/photo-15.png)

### **图片上写字**

在图片上写字是期望实现通过MidJourney实现隐写，即将文字隐藏在图片中，并通过MidJourney再读取其中的文字。从技术原理上，这样显然不现实，因此笔者尝试将中文、英文和数字呈现在图片中。

> text of "子午" engraved on bricks wall

![](images/2023/05/photo-16.png)

> text of "great wall" engraved on bricks wall

![](images/2023/05/photo-17.png)

> text of "9527" engraved on bricks wall

![](images/2023/05/photo-18.png)

结果依然是不理想，无论是中文、英文还是数字，都无法按照Prompt的描述将文字体现在图片中，但实现这个效果并不难，结合NLP技术和现有的图像生成应当在未来可以实现理想的效果。

### **技术型图片生成**

有没有可能通过MidJourney生成技术类的图片呢？比如生成一张3D版本的网络拓扑图。

> A topology diagram with routers, switches, firewalls, DMZ zones, and server devices,3d rendering

![](images/2023/05/photo-19.png)

显然，对于技术类型图片的处理和生成只会其意而无法产生真正有意义的、严谨的技术图片。于是，将Prompt做了更加细化的描述：

> A topology diagram shows a firewall followed by a switch, a switch connected to a DMZ and an office zone, a web server in the DMZ, and three employee office computers in the office zone.

![](images/2023/05/photo-20.png)

同样，生成的图片在技术呈现上毫无意义甚至错误百出的，且字母或描述都出现了乱码和错误，无论重新生成多少次，最终的结果都不太理想，远看像一回事，近看无法直视。最后，附上笔者用于处理卡通3D头像的Prompt：

> personality girl,super detail,soft colors,soft ighting,anime,high detail,art station seraflurart,ip,divine , cinematic edge lighting,light anddark contrast,Delicate features,Soft light,clean background.IP design,art bv Studio Ghibli,3d,c4d,blender,UnreaEngine,8kOC rendererbest quality --iw 1.5 --s 750

使用上面的Prompt需要根据性别修改girl或boy，原图需要尽可能高清和干净，产生的效果显著，如果不像原图，需要调整--iw参数，并多生成几次：

![](images/2023/05/photo-21.png)

### 更新
目前的MidJourney已经支持图片中写字的功能，但仅限于英文字母和数字。