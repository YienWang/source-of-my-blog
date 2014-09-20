---
layout: post
title: "About Network Perfomance"
date: 2014-08-30 00:43:59 +0800
comments: true
categories: android network-performance
---
今晚是阿里的在线笔试，又是那道熟悉的网络优化题。这次再回想一下自己基本写了些什么：
>1. 合理的合并一些网络请求，减少请求次数。
2. 服务端针对客户端屏幕分辨率返回恰当大小的图片。
3. 使用SPDY和WebP技术优化。前者提供网络通讯协议的优化，后者提供图片流量的优化。
4. 根据Http中Cache-Control做好本地缓存。
5. 当内容为文本类型时，使用Json作为传输内容，同时开启GZip压缩。
6. 具体一点的场景：在列表滑动过程中，暂停对数据的请求，在滑动结束后才对内容进行请求（主要针对图片）。这在一定一定程度上减少了线程并发数，使主线程能够流畅地执行（保证了界面的响应度）
<!-- more-->
讲了几个点，这几个点基本上算是自己看过一些书，看过一些视频，还有自己遭遇过类似情况之后的总结吧。 虽然有些点，我也确实无法说出个所以然。不过现在既然说到了，作为一个整理控，不好好整理一下，怎么能行~！

#### 1.合理地合并一些网络请求  
这一点是在之前开发SmartLife的时候总结出来的。  
写服务器代码的那个~~渣渣~~同学按照之前写网站的经验，弄了几个API。这几个API基本是这样子的：
1. 获取用户所有的活动id 
2. 根据id获取活动的具体详情。

如果我是在写前端，后台这么做我还可以忍（自己做的网站，基本上也就不考虑QPS多少了，反正还没多到不能接受），一个一个请求就一个请求咯。但问题是在手机上就没法这么做了啊。
**一个是流量没法接受（个人活动有多少啊，每个个人活动还要再发请求，流量简直要命）。其次是一个一个请求，就算是并发也太慢啊，还要考虑多线程，并发，线程安全，头都大了。**
就这么跟写后台的那位那么说，然后那货竟然说*谁管你啊，能请求就好了，改来改去烦死了*。就一个for循环搞定的事竟然被这么说，当时就怒了。那种后台代码简直不能看好吗，没让他重写都算是好的了。  
好像跑题了- -总之后来终于让他给了个for循环。我本地缓存数据还不行么。

#### 2. 服务端针对客户端屏幕分辨率返回恰当大小的图片
这点的话主要是针对服务器那边的了（手机这边倒也没什么，缩放是正常就要做的）。手机这边能够将手机分辨率作为参数发送给服务器，服务器根据屏幕分辨率提前处理图片并返回适当大小的图片。  
获取屏幕分辨率的方法~  
(实际上，直接将控件的高度反馈给服务器也可，但是考虑到通用性，更加适合的方式应该是返回一个屏幕密度给服务器，服务器根据这个范围来返回对应大小的图片）
```java
 DisplayMetrics metrics = new DisplayMetrics();
 getWindowManager().getDefaultDisplay().getMetrics(metrics);
 metrics.density;
```
见备注有DIP及DP的分类，计算。

#### 3. 使用SPDY协议和WebP
SPDY是google推出来的一个网络协议，似乎类似于TCP层，能够大量减少网络访问时间：
>SPDY is a replacement for HTTP, designed to speed up transfers of web pages, by eliminating much of the overhead associated with HTTP. 

这里提到的点是**Web Page**，但实际上根据SPDY的原理，在手机上也有很大的使用空间。之前有见过一个Android的网络请求库就实现了SPDY协议，具体见[okhttp][okhttp]。


#### 4. 根据Http中Cache-Control做好本地缓存。


#### 5. 使用Json和协议开启GZip

### 备注
##### DIP 屏幕无关像素
> density 密度
The logical density of the display. This is a scaling factor for the Density Independent Pixel unit, where one DIP is one pixel on an approximately 160 dpi screen (for example a 240x320, 1.5"x2" screen), providing the baseline of the system's display. Thus on a 160dpi screen this density value will be 1; on a 120 dpi screen it would be .75; etc.
This value does not exactly follow the real screen size (as given by xdpi and ydpi, but rather is used to scale the size of the overall UI in steps based on gross changes in the display dpi. For example, a 240x320 screen will have a density of 1 even if its width is 1.8", 1.3", etc. However, if the screen resolution is increased to 320x480 but the screen size remained 1.5"x2" then the density would be increased (probably to 1.5). --[display martrix][dp]

屏幕的逻辑密度，用于将DIP与px进行转换的一个缩放系数。在160dpi(240x320) 的手机上，该系数为1，即1px = 1dp。

支持SPDY协议的客户端 [okhttp][okhttp]
官方网站上的说明 [SPDY Performance on Mobile Networks ][spdy-for-mobile]

[okhttp]: http://square.github.io/okhttp/
[spdy-for-mobile]: https://developers.google.com/speed/articles/spdy-for-mobile
[dp]: http://developer.android.com/reference/android/util/DisplayMetrics.html