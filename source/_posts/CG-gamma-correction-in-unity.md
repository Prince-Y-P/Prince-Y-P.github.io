---
title: 【渲染】理解伽马校正(Gamma Correction)与Unity的Gamma or Linear workflow
date: 2020-09-14 00:11:00
categories:
- CG&Rendering
tags:
- CG
- Rendering
catalog: true

---

By Prin@UWA

笔者搜集阅读了一些与Gamma校正(Gamma Correction)、sRGB颜色空间等概念相关的资料，在学习相关知识时，由于理解的错误踩了很多坑。本文结合自己的理解，对相关理论以及在Unity中的处理方式进行了整理，希望能帮助刚接触这个知识的同学有一个粗浅的理解。

# 伽马校正（Gamma Correction）
## 存在的原因
对于Gamma校正存在的原因，在网络上略有争议。汇总各种资料，主要存在以下两种观点：
1. 历史原因：去的CRT显示器屏幕上显示的颜色对于传递而来的原始值并不是线性的
2. 人眼对自然亮度感知是非线性的


笔者认为，以上两种因素是同时存在且不冲突的，都是Gamma校正产生的原因。以下来自[kinematicsoup官网](https://www.kinematicsoup.com/news/2016/6/15/gamma-and-linear-space-what-they-are-how-they-differ)的说法笔者比较认同：

> The need for gamma arises for two main reasons: The first is that screens have a non-linear response to intensity. The other is that the human eye can tell the difference between darker shades better than lighter shades. This means that when images are compressed to save space, we want to have greater accuracy for dark intensities at the expense of lighter intensities. Both of these problems are resolved using gamma correction, which is to say the intensity of every pixel in an image is put through a power function. Specifically, gamma is the name given to the power applied to the image.

[Wikipedia-Gamma correction](https://en.wikipedia.org/wiki/Gamma_correction)中谈到：
> Although gamma encoding was developed originally to compensate for the input–output characteristic of cathode ray tube (CRT) displays, that is not its main purpose or advantage in modern systems.

由此基本可以明确：CRT显示器的历史原因是Gamma Correction的起源，与人眼感知能力等相关的其他因素是Gamma Correction一直延续下来的原因之一。 因此，不能绝对地主张其中一种原因而排斥其他说法。以下是笔者搜集和理解到的对两种原因的解释。


### CRT显示器特性的原因
此处主要借鉴冯乐乐女神的文章。
在早期，CRT几乎是唯一的显示设备。但CRT有个特性，它的输入电压和显示出来的亮度关系不是线性的，而是一个类似幂律（power-law）曲线的关系。也就是说，使用一个电压轰击CRT Monitor屏幕上的一种图层，这个图层就可以发亮，我们就可以看到图像了。但是，人们发现，咦，如果把电压调高两倍，屏幕亮度并没有提高两倍啊！
典型的CRT显示器的伽马曲线大致是一个伽马值为2.5的幂律曲线。显示器的这类伽马也称为**display gamma**。

这个幂律曲线的公式可以简单表述为下式：

$ \gamma(u) = Au^{\gamma } (u\in \left [  0, 1\right ],\gamma(u) \in \left [  0, 1\right ] )$      **where u is R, G, or B**

一般情况下取A=1，那么，输入一个颜色值(0.5, 0.5, 0.5)，由该公式得到的输出为(0.177, 0.177, 0.177)。也就是说，显示器的Gamma转换会使输入的像素颜色的亮度变低（Darker）。

由于这个问题的存在，那么图像捕捉设备就需要进行一个**伽马校正(Gamma Correction)**，对显示器的伽玛变换进行**反向补偿**(变亮)。校正使用的伽马叫做**encoding gamma**。
所以，一个完整的图像系统需要2个伽马值：
- encoding gamma：它描述了encoding transfer function，即图像设备捕捉到的场景亮度值（scene radiance values）和编码的像素值（encoded pixel values）之间的关系。
- display gamma：它描述了display transfer function，即编码的像素值和显示的亮度（displayed radiance）之间的关系。

显示系统的处理流程如下图所示：
![Alt text](./DiaplaySys.png)

[PZZZB](https://zhuanlan.zhihu.com/p/66558476)的图较好地诠释了对应不同Gamma值的幂律曲线之间的关系：
![Alt text](./power-law.jpg)


encoding gamma和display gamma的乘积就是真个图像系统的end-to-end gamma。如果这个乘积是1，那么显示出来的亮度就是和捕捉到的真实场景的亮度是成比例的。如果我们没有用一个encoding gamma对shader的输出进行校正，而是直接显示在屏幕上，那么由于display gamma的存在就会使画面失真。
虽然现在CRT设备很少见了，但为了保证这种感知一致性（这是它一直沿用至今的很重要的一点），同时也为了对已有图像的兼容性（之前很多图像使用了encoding gamma对图像进行了编码），所以仍在使用这种伽马编码。

至此可以大致理解对于显示器的Gamma校正的原理。然而，实际问题往往更复杂一些，此处做一个简要提示，感兴趣的同学可以深究。
encoding gamma和display gamma的乘积为1的话，可以让显示器精确重现原始场景的视觉条件。但是，原始场景的观察条件和显示的版本之间存在两个差异我们需要乘积不是1的end-to-end gamma，来保证显示的亮度结果在感知上和原始场景是一致的。根据《Real-time Rendering》一书中，推荐的值在电影院这种漆黑的环境中为1.5，在明亮的室内这个值为1.125。
> 个人电脑使用的一个标准叫sRGB，它使用的encoding gamma大约是0.45（也就是1/2.2）。这个值就是为了配合display gamma为2.5的设备工作的。这样，end-to-end gamma就是0.45 * 2.5 = 1.125了。


### 人眼感知能力的原因
人眼对亮度的感知是非线性的。换句话说，**亮度上的线性变化在人眼看来是非均匀的**。从0亮度变到0.01亮度，人眼是可以察觉到的，但从0.99变到1.0，人眼可能就根本差别不出来，觉得它们是一个颜色。人眼对暗部的变化更加敏感，而对亮部变化其实不是很敏感。
通过知乎上韩世麟答主分享的[视频](https://www.zhihu.com/question/27467127#answer-10413243)可以对这个问题有一个直观的理解。

![Alt text](./灰阶关系0.5.jpg)

以下两比较通俗的句话可以帮助读者对这个问题进行理解：
* 人心目中（美术中的）看起来中灰的色块，其物理亮度值大约在白色块的20%左右。
* 自然界的0.2，在心目中的地位是0.5。

现实生活中的例子如下图所示：
![Alt text](./颜色卡.jpg)

而如果将纯白色块与纯黑色块进行1比1混合，得到的结果在人眼看来是浅灰色。

符合人的直觉的灰度(或者理解为亮度)均匀变化的情况如下图：
![Alt text](./人眼感知.png)
们看到这张图，会自然而然的认为中间的地方即灰度为0.5的地方。事实上，在线性颜色空间下，也就是实际人看到的中间部分的灰度值约为0.218。

那么，美术中使用的中间**灰色**应该设置像素值为多少呢？计算机中图片上的值与最终人们看到颜色是如何对应的呢？ 这也是曾一度困扰笔者的一个问题。
在Photoshop中作图，利用颜色取样工具，会发现上图的中间灰色对应的像素颜色值就是(127, 127, 127)，换算成0到1区间的表示方式即约为(0.5, 0.5, 0.5)。由此可见，美术要绘制出直觉上介于黑白中间的灰色，把灰度值设置为0.5就可以了。
上文中已经阐述过，显示器会对输入的颜色值进行display gamma转换。也就是说，输入给显示器的灰度值0.5，经过显示器的转换之后，会变为0.218，而这个0.218的灰度，又刚好符合人眼对于中等灰度（0.5）的直觉。这是一个有趣的巧合。这也可能是显示器的Gamma转换一直延续的原因。

## Gamma的定义
[wikipedia - Gamma correction](https://en.wikipedia.org/wiki/Gamma_correction)对Gamma校正的相关概念进行了清楚的定义。
**Gamma correction**, or often simply **gamma**, is a nonlinear operation used to encode and decode luminance or tristimulus values in video or still image systems.
Gamma correction is, in the simplest cases, defined by the following power-law expression:
${\displaystyle V_{\text{out}}=A{V_{\text{in}}^{\gamma }}}$

不同的文献资料对颜色空间相关转换的称呼不统一，这也是这块知识不容易理解清楚的原因之一。为此，笔者对同一过程的不同表述方式进行了整理，以便读者进行理解。

$\gamma <1$时(即一些资料所说的1/2.2)，该值称为**encoding gamma**，该幂律函数进行的非线性转换称为**gamma compression**，或**gamma encoding**。该转换会把颜色的亮度变大，因此，这个过程也叫做**补偿**，狭义上的**伽马校正**。

$\gamma > 1$时(典型的范围在2.0到2.4之间)，该值称为**decoding gamma**，该幂律函数进行的非线性转换称为**gamma expansion**，或**gamma decoding**。该转换会把导致颜色的亮度变低(Darker)，显示器的**伽马转换**就是这样一个过程。

# 颜色空间(sRGB & Linear RGB)

> Unity中的Color Space可以设置为Linear和Gamma，但这一说法会有争议。有人会说不存在Gamma Color Space的定义，Wikipedia上对Color Space的介绍中也确实没有提到Gamma Color Space。Gamma Color Space可能算是Unity自己提出的一个概念，指的是将图片的像素经过伽马校正后的颜色空间。
> Unity文档中提到：**The accepted standard for gamma space is called sRGB**。准确地说，sRGB才是一种Color Space。为了方便理解，读者可以认为gamma space就是sRGB color space。

## sRGB
[Wikipedia](https://en.wikipedia.org/wiki/SRGB)上定义：
sRGB (standard Red Green Blue) is an RGB color space that HP and Microsoft created cooperatively in 1996 to use on monitors, printers, and the Web.

我们可以粗浅地理解为：
**sRGB格式相当于对物理空间的颜色做了一次伽马校正(Gamma Correction)**。sRGB空间下的颜色即为经过伽马校正后的颜色。将sRGB格式的Texture输入给显示器，显示器经过自身的Gamma变换之后，输出的值就是准确的、线性空间下的颜色值，这样人从显示器看到的图像就和人眼直接观察物理世界一样了。

> 笔者认为，由此我们很容易理解为什么Gamma校正和转换的过程被称为**Encoding**和**Decoding**——sRGB是一种格式，转换成这种格式我们可以叫做：**编码成sRGB格式**，那么其逆运算的过程就是**解码**。

wikipedia中介绍了不同颜色空间(CIE XYZ to sRGB)的转换过程。我们只需要理解**线性空间**到**sRGB空间**的计算公式即可。
The following formula transforms **the linear RGB values** into **sRGB** :

![Alt text](./1599332150051.png)
* where u is R, G, or B.



# Unity对Color Space的处理
前面已经阐述过，显示器在显示的时候，会用display gamma把显示的像素进行display transfer之后输出实际显示的亮度值。所以，我们要在这之前，对图像先进行gamma encoding(或Gamma校正)。也就是说，我们输出给显示器的颜色值，应当是sRGB空间下的值，这样，显示器输出给人眼的值，才是我们期望人眼看到的值（和人眼直接观察物理世界一样）。

Unity官方文档解释到：
The reason both gamma and linear color spaces exist is because **lighting calculations should be done in linear space in order to be mathematically correct**, but the **result should be presented in gamma space to look correct to our eyes**.
如果要确保计算机渲染的效果最大程度接近真实世界，我们应当在线性空间下进行光照计算，因为这符合真实世界的规律（使用贴图中颜色的Linear RGB值进行计算）；应当输出给显示器Gamma空间的颜色值（输出给屏幕sRGB格式的像素）。

> Unity文档中的以下表述可以帮助读者更好地理解颜色空间的问题：
> Even though **monitors** today are digital, they still take a **gamma-encoded signal as input**. Image **files** and video files are explicitly encoded to be in gamma space (meaning they **carry gamma-encoded values**, not linear intensities). This is the standard; everything is in gamma space.

接下来我们看看Unity对Color Space的两种workflow：Gamma和Linear。
> 该设置在Unity的Edit -> Project Settings -> Player -> Other Settings中。

其实说将Color Space设置成Gamma和Linear容易造成误解，因为这个设置并不是说输出的值是Gamma空间还是Linear空间。该设置只是选择两套处理颜色空间的方案(Workflow, or Pipeline)。
![Alt text](./An+image+comparing+gamma+and+linear+pipelines.png)


## Gamma workflow
 当选择Gamma Space时，实际上就是“放任模式”，不会对shader的输入进行任何处理，即使输入可能是非线性的；也不会对输出像素进行任何处理，这意味着输出的像素会经过显示器的display gamma转换后得到非预期的亮度，通常表现为整个场景会比较昏暗。

我们使用Photoshop创建两张颜色为(127, 127, 127)纯色图片（映射到0到1的数值为0.498）。在Unity中将图片设置为UGUI的Image，并在屏幕上显示。在Import Settings当中，将其中一张图片勾选为sRGB(Color Texture)，另一张图片不勾选该项。
由下图可见，两张图片在显示上并没有区别，输出给显示器的颜色值都为0.498。

![Alt text](./1.png)
整个gamma workflow的过程中不会进行gamma校正。

##  Linear workflow
选择Color Space为Linear的含义，并不是输出给屏幕的颜色为Linear RPG值，而是说，进行的光照计算的处理，是在Linear Space下进行的。下段文字为kinematicsoup介绍的Linear Pipeline的过程:
The input colors and textures have their gamma correction removed before shading, putting them into linear space. When shaded, the result is physically correct because the shading process and inputs are all in the same space. After, any post effects should be computed while the frame is still in linear space, as post effects are typically linear, much like shading. Finally the image is then gamma corrected so it will have the proper intensity after the display’s gamma adjustments.
在Linear workflow下，Unity会对sRGB格式的Texture进行decoding，将sRGB空间的颜色值转换为线性空间，然后在Linear Space下进行着色（光照计算等）。在最后输出阶段，进行的伽马校正（gamma correction），将Linear空间的像素转换为sRGB空间，输出给显示器。经过显示器的转换之后，输出给人眼的颜色就是更接近真实的颜色（亮度）。

在Linear workflow下，做前文相同的操作，可以发现，没有勾选sRGB的Texture（右边）会更亮一些。
![Alt text](./QQ截图20200906041147.png)
初始状态下，两张图片的亮度都为0.498。

左边的Texture勾选了sRGB，那么Unity会把这张图片当作sRGB空间下的图片，在进行着色前，先将图片进行Decoding(Remove Gamma Correction)，转换成Linear Space下的图片，在输出给显示器之前，再进行Encoding，编码成sRGB格式。此时，输出给显示器的亮度值经过互为逆运算的两个步骤，仍然为0.498。经过显示器的转换，即为我们看到的左边的方块。

右边的Texture没有勾选sRGB，Unity会把这张图片当作Linear Space下的图片，也就不进行Decoding。在着色之后，进行Encoding(Gamma Correction)，输出给显示器的亮度值为0.733。经显示器转换处理之后，看到的就是右边的方块。

经过笔者计算验证，Unity进行Encoding的时候，使用的公式就是上文提到的公式（可能存在少量误差）：

![Alt text](./1599332150051.png)
* where u is R, G, or B.

# 最后
促使笔者对本文的知识点进行整理的原因，主要是本公司（UWA）初始版本的的特效检测工具在Linear Color Space下存在计算得到的OverdrawRate偏高的小BUG。详情请看UWA问答：https://answer.uwa4d.com/question/5f51b8ba9424416784ef20d7

# Ref
https://blog.csdn.net/candycat1992/article/details/46228771
https://www.zhihu.com/question/27467127#answer-10413243
[Unity Manual](https://docs.unity3d.com/Manual/index.html)
https://www.jianshu.com/p/e15932c40bea
https://en.wikipedia.org/wiki/SRGB
https://en.wikipedia.org/wiki/Gamma_correction
https://unity.cn/projects/unite-2018-qian-tan-jia-ma-he-xian-xing-yan-se-kong-jian
https://www.kinematicsoup.com/news/2016/6/15/gamma-and-linear-space-what-they-are-how-they-differ
https://zhuanlan.zhihu.com/p/66558476
