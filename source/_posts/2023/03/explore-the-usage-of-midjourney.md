---
title: 探索MidJourney的用法
date: 2023-03-30 02:26:35
author: repoog
excerpt: MidJourney最近非常火热，通过文本生成的图片极其逼真，笔者尝试基于官方的操作指令探索MJ的更多用法和效果。
comments: true
tags:
  - AI
  - AI作图
  - MidJourney
  - 卡通图片
categories:
  - 人工智能
---

Midjourney是一个人工智能程序，由位于旧金山的独立研究实验室Midjourney, Inc.创建，使用自然语言描述中生成图像，称为 "prompt"，于2022年7月12日发布公测版本。Midjourney团队由David Holz领导，同时David也是Leap Motion（一款动作感知设备）的联合创始人。用户只需要注册Discord（访问需科学上网），并授权MidJourney授权访问，即可向MidJourney机器人发送指令用创作艺术品。

MidJourney的免费用户可以生成25个作品，之后需要订阅付费版本才能继续使用，包括基础版、标准版和专业版。可以将MidJourney机器人添加到自己的频道使用，也可以直接给机器人发送指令，前者可能会受到其他人指令和返回结果的干扰。

通过给机器人发送指令，每次返回结果默认是四张照片，通过点击U（Upscale）和V（Version），可以选择某个版本继续生成或者直接生成最终的图片，且历史产生的所有图片都会在MidJourney官方的个人账户中呈现，并可以下载图片，或查看图片的生成指令、Prompt，亦或者将图片设置为账户头像。

MidJourney的使用和生成取决于Prompt以及参数设置，即发送给机器人的指令，越是详细的Prompt越能够产生自己预期的结果（需用英文描述），而参数设置可以用于调整生成的效果，常见的指令、参数及含义如下：

*   /imagine：使用Prompt生成图片；
*   /settings：设置默认的模型版本、图片质量、图片类型和生成模式等；
*   \--aspect/--ar：设置生成图片的尺寸比例，比如1：1，5：4，16：9等；
*   \--chaos/--c <0-100>：设置图片的混沌值，数值越大，图片的不可预期性越大；
*   \--no：用于Prompt中排除的关键词，多个关键词用逗号分隔；
*   \--quality/--q <.25, .5, 1, 2>：图片渲染的质量（默认是1），数值越大，渲染时间越长，质量越好；
*   \--seed <0-4294967295>：图片生成用的随机值，相同的随机值可以产生结果类似的图片，可以用于图片微调；
*   \--stylize ：图片结果的艺术化程度，数值越大，艺术性越高，与Prompt关联性越小；
*   \--iw <0-2>：如果Prompt中有图片链接，该参数用于设置最终图片参考图片链接的比重，默认是0.25；

总而言之，MidJourney堪比马良的神笔，只要能够准确描述图片内容、类型、风格、镜头、色彩、材质、心情等等信息，便能够产生对应的图片。

### **凭空生成图片**

MidJourney基于Diffusion的模型擅长凭空生成图片，通过Prompt描述生成图片，但如何写出好的Prompt本身也是难题，因此类似[https://prompt.noonshot.com/](https://prompt.noonshot.com/)这样专门用于生成Prompt的网站应运而生。

可以从配置中看到，图片类型、照明、镜头、艺术风格、颜色、材料都可以通过Prompt进行设定，Prompt描述越精准，生成的图片越精确，比如：

![The sun is just rising in the early morning, the jungle on the island is on the left, the middle and right is an endless sea, between the sea and the jungle is the beach, at the foot is a piece of grass protruding from the island, the grass is not far from the cliffs intersecting with the sea, and sea birds are hovering over the beach.:: photorealistic::2 natural lighting::1 --ar 16:9  --v 5](images/2023/03/mj-01.png)

> The sun is just rising in the early morning, the jungle on the island is on the left, the middle and right is an endless sea, between the sea and the jungle is the beach, at the foot is a piece of grass protruding from the island, the grass is not far from the cliffs intersecting with the sea, and sea birds are hovering over the beach.:: photorealistic::2 natural lighting::1 --ar 16:9  --v 5

上图中Prompt的photorealistic和natural lighting后的数字用于调整该指令的强度，数字越大，效果越明显。

纯描述性的Prompt也可以生成意想不到的图片，比如：

![At the seaside on a moonlit night, waves crash onto the beach while the big, round moon hangs in the sky, accompanied by numerous shining stars. In the distance, a military ship is visible with twinkling lights, and nearby a few dolphins jump out of the ocean surface.](images/2023/03/mj-02.png)

> At the seaside on a moonlit night, waves crash onto the beach while the big, round moon hangs in the sky, accompanied by numerous shining stars. In the distance, a military ship is visible with twinkling lights, and nearby a few dolphins jump out of the ocean surface.

如果熟悉摄影和相机，甚至可以在Prompt中加入镜头信息和相机设备信息，比如24mm（广角镜头）、F10（小光圈）、Nikon D800（尼康D800）。

相同的Prompt每次生成的图片不一定都一致，需要通过重新生成按钮来变换生成的结果，选择自己喜欢的那张。

![](images/2023/03/mj-03.png)

![](images/2023/03/mj-04.png)

![](images/2023/03/mj-05.png)

### **Logo设计**

凭空生成的图片的一种细分作用是可以用于商业创作，其中包括Logo设计。使用MidJourney进行Logo设计几乎只需要说清楚，就能够产生理想的效果，同时可以在Prompt中标明想要的Logo设计类型，比如字符型（lettermark）、吉祥物（Mascot）、徽章（Emblems），真正的What you can say, what you can see.

比如笔者为站点设计（生成）的Logo：

![](images/2023/03/mj-06.png)

> a simple,vector lettermark logo of R, the log is for a website called 'SideWindow',its theme is hacking and managment

又比如：

![](images/2023/03/mj-07.png)

> a simple,vector lettermark logo of a website called 'SideWindow',its theme is technology,cybersecurity and development,managment

### **人物照片上妆**

通过Prompt对于原图人物的妆容描写，也可以改变人物的妆容，看到化妆后的效果。

![](images/2023/03/mj-08.png)

> <Photo URL> is a boy with black glasses --iw 1.5 --v 5

![](images/2023/03/mj-09.png)

> short black hair,red lips,big eyes --v 5

以上两张图均是以笔者儿子身份照片为例生成的，一张是戴黑色眼镜的效果，一张是留短发、涂口红、大眼睛的效果，虽然脸型略有不同，但和原图人物相差也不是很大，可以看到改变妆容之后的样子。

### **照片生成卡通**

根据真人照片生成卡通模式是消耗最多Job（作业）次数和时间（Basic Plan每个月只有200min的快速生成时间）的，许多人也在寻找如何将真人照片生成卡通模式。根据真人照片生成卡通人物形象的图片，只需要在Prompt中写诸如cartoon style或者cartoon character即可。比如：

![](images/2023/03/mj-10.png)

> <Photo URL> cartoon style,big smile,24-70mm --ar 16:9 --iw 1 --v 5

![](images/2023/03/mj-11.png)

> <Photo URL> cartoon character,happy --ar 16:9 --v 5

上面的卡通人物是根据全家福照片生成的，可爱但很怪异，最大的问题是：不像，完全不像我们家任何一个人。

经过N次Prompt和参数的变换，笔者发现基于人物照片生成人物图片的共性问题：

* 成图与原图中的人物人数、姿态、动作会有很大差异，甚至出现怪异的表情和动作；
* 对于国人照片，生成图片的人物的风格和长相普遍偏向东南亚或欧美亚洲人的风格；
* 直接使用cartoon style等Prompt的设定无法产生原图对应的卡通版本；
* 原图人数越多，成图的效果和预期相差越远、越大；
* 需要尽可能精准的Prompt描述原图中的人物、动作、形态才能更接近原图的卡通版本；

即便在参数--iw中设置了原图比重最高：

![](images/2023/03/mj-12.png)

> <Photo URL> cartoon style --iw 2 --v 5

好在MidJourney提供了Remix模式，该模型可以提取原图的结构信息用于生成新的图片（take the general composition of your starting image and use it as part of the new Job）。默认情况下Remix是关闭状态，因此需要通过/settings命令或/prefer remix命令打开该模式。

通过Remix模式生成卡通版本或其他版本的图片，比非Remix模式要好很多：

![](images/2023/03/mj-13.png)

> cartoon style --iw 2 --no glasses --v 5

可以看到生成的图片还是会偏向欧美人长相，因此需要添加比如Chinese等限定词到Prompt中，并通过--no参数来排除系统自动添加的眼睛、帽子等装饰。

![](images/2023/03/mj-14.png)

> <Photo URL> cartoon style --v 5

![](images/2023/03/mj-15.png)

> <Photo URL> cartoon style --iw 2 --no glasses --v 5

上图的结果已经和原图非常相像了，但产生这样效果的图片还需要多次“重新生成”，相同的Prompt不会每次产生一样的结果，因此需要多次尝试生成才能看到理想的效果。所以如果看到网上别人的成图与自己的成图不一致，也不必气馁，试着调整参数或者多生成几次。

要生成和原图相像的卡通效果的图片，还可以采用以下办法增强效果：

* 尽可能使用单人照片；
* 在Prompt中添加多张同一个人的照片；

在所有图片的参数中笔者都仅使用默认的--q参数值，以节省时间，即便如此，也仅用了两天多时间便将首月的200min全部用完。还有很多其他使用MidJourney生成图片的有趣思路和想法待验证：

* 多人照片合影；
* 真人照片换装；
* 真人照片换背景；
* 照片局部修改（MidJourney预计6月份推出）；
* 技术型图片生成（如网络拓扑图）。