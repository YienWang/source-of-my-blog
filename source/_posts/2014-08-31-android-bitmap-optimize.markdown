---
title: "Android Bitmap 和 OOM"
date: 2014-08-31 12:42:22 
comments: true
tags: [android,Bitmap] 
---
关于Android的图片格式，似乎有很多东西可以讲的。基本上一讲到面试，就会问到怎么处理大图片。  
刚在群上看到一个问题：*“在加载一张10M图片的时候要如何避免OOM?”*  
这个问题，感觉还是蛮抽象的，因为10M的图片，没有说格式，没有说大小，没有说图片怎么展示，这就是个很宽泛的问题啊。

<!--more-->

1. **图片格式**
---------
其实图片的格式关系不大，在读入Android的中时候都是使用bmp格式来显示的。而这些bmp所占用的内存，则是由**“像素个数*单个像素所占字节数”**来决定，如此一来读入时所占的内存其实跟图片格式一点关系都没了。  
关于Bitmap，其Bitmap.Config会影响单个像素所占字节数，因此选择合理的Bitmap.Config也能够减少OOM现象的发生。

2. **图片尺寸过大，但需要全部展示**
-------
这种情况下，直接读入内存势必是会导致OOM的了。所以在对整张图片进行**完全**decode之前，先要对比图片的原大小跟目标的大小，在设置合理的下采样系数。 `BitmapFactory.Options` 关键属性:
`inJustDecordBound` 在只需要读取图片的宽高时设置。在真正需要decode的时候记得设置为false，否则decode返回出来的Bitmap会是null。  
`inSampleSize` 下采样率，数字越大，图片越小。通常设置会是2的倍数~~（怀疑是为了兼容OpenGL ES1.0）~~

3. **图片尺寸过大，根据需要来展示其中某部分**
---------
这常见于加载整张大地图的情况：在大图模式下展示全局，同时可以通过手势来对画面进行缩放。  
这种情况无法单纯地使用采样来做，因为在局部模式下，显示的只是部分内容，inSamplesize不会太大，那么对整张图片进行decode势必会OOM。这个时候就需要能够只对图片指定部分进行decode的工具了。  
Android提供了一个叫做 `BitmapDecodeRegion` 的工具来实现这种局部加载的功能。

**补充**
=============
### **Bitmap.Config**
Android的Bitmap.Config有如下几种格式，其实看名字就可以知道具体的含义了。按照ARGB的次序，只有1位代表是透明通道，刚好3位是RGB，4位就是ARGB了。对应的数字代表‘位数’，即bit。比如 `RGB_565` 就是红色5位，绿色6位，蓝色5位。

1. ARGB_8888  
默认的格式。每个像素使用4byte来存储。对于一张3000x4000的图片而言，所占内存大小就是12M。
2. RGB_565
每个像素使用2byte来存储，不存储透明通道。跟上面一样的一张图片可以节省一半空间。但是显示效果较差，通常需要设置 `inDither` 来插值。

>  To get better results dithering should be applied. This configuration may be useful when using opaque bitmaps that do not require high color fidelity.

3. ALPHA_8  
跟RGB相反，只存储透明通道，因而只占1byte空间。可以用来作为蒙版。蒙版中的黑色代表擦除。

> This is very useful to efficiently store masks for instance.

4. ARGB_4444  
每个像素使用2byte来存储，且每个通道均保留同样的梯度。在API13的时候deprecated，原因是显示效果不佳。建议使用8888
