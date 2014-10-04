title: Android如何省电
date: 2014-10-04 23:43:10
tags: [android,省电]
---
前言
-------------
问题的来源是网易的一次面试，虽然说我觉得问本科生这种问题确实有点奇怪= =在网上一直没找到好的资料，所以还是自己总结一下了。
<!--more-->
正文
----------------


补充
------------
### RTC和RTC_WAKEUP
两者的区别就是，后者会唤醒设备，而前者到了指定时间如果设备不是唤醒状态就不会有反应,直到设备唤醒。  
在AlarmManger的官方文档中说AlarmManger在唤醒时持有了wake lock，因而CPU不会进入睡眠状态，但是当广播事件的`onReceive()`结束时，锁也就释放了。如果在这个过程中试图创建一个后台服务，很可能在这个服务启动的时候CPU又继续睡眠了。所以需要控制好wake lock的持有问题。

### 手机休眠会终止线程执行吗
看到这样的一个问题[When will android stop its cpu without wake lock?][2]  
如果自己没有手动地获取wake lock且没有其他wake lock是激活状态的，那么CPU就会进入睡眠状态，所有的线程都不会有机会执行。难怪自己的定时器偶尔会停下来不动

资料
--------------
[AlarmManger][1]
[PowerManager.WakeLock][4]
[开发者分享:如何让应用更省电][5]
[Android：如何在传输数据的时候花更少的电][6]
[Android手机耗电深度解析：3G耗电是WiFi四倍][3]
[sony:Reducing power consumption of connected apps][7]

[1]: http://developer.android.com/reference/android/app/AlarmManager.html
[2]: http://stackoverflow.com/questions/8337418/when-will-android-stop-its-cpu-without-wake-lock
[3]: http://digi.tech.qq.com/a/20131115/014966.htm
[4]: http://developer.android.com/reference/android/os/PowerManager.WakeLock.html
[5]: http://it.sohu.com/20130806/n383502784.shtml
[6]: http://developer.android.com/training/efficient-downloads/index.html
[7]: http://developer.sonymobile.com/2010/08/23/android-tutorial-reducing-power-consumption-of-connected-apps/
[8]: http://my.oschina.net/zhangjie830621/blog/122116