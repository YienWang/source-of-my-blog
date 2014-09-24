title: Android尺寸
date: 2014-09-22 13:37:16
tags: [android,参数]
---
Android的绘图单位：
--------------
DIP/DPI/PX：
> The logical density of the display. This is a scaling factor for the Density Independent Pixel unit, where one DIP is one pixel on an approximately 160 dpi screen (for example a 240x320, 1.5"x2" screen), providing the baseline of the system's display. Thus on a 160dpi screen this density value will be 1; on a 120 dpi screen it would be .75; etc.  
> -[DisplayMetrics][displaymetrics]

看看官方的说法，然后你就知道了。


[displaymetrics]: http://developer.android.com/reference/android/util/DisplayMetrics.html