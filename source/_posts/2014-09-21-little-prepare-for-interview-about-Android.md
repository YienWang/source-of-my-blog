title: Android面试前的一点准备
date: 2014-09-21 07:55:20
tags: [android, 面试前, 三句说清]
---
关于几个容易忘记的概念性常识。不定期更新
1. **LaunchMode**
2. **Process分级**
3. **SurfaceView**  
4. **Touch事件分发** *---14.9.21*
5. **Java线程**

<!--more-->
LaunchMode
------------------
>LaunchMode是Android的Activity组件启动的一个设置。它有`standard`，`singleTop`，`singleInstance`，`singleTask`四种模式。每种模式都有自己适合的场景。


Process分级
-----------------
> Android的Process的级别共五种。这五种分别是前台进程，可视进程，服务进程，后台进程，空进程。在运存空间不足时，Android会按上面的等级依次回收进程。

SurfaceView
------------
>SurfaceView是Android提供的，以另外一个线程刷新界面的View控件。SurfaceView使用单独的线程进行画面的刷新而不会阻塞主线程，非常适用于用于大量绘图的场景。同时SurfaceView的刷新周期用户可控,方便调节FPS。

之前YL问过我SurfaceView的问题，当时综合了几篇文章，写了这个：
> 1. SurfaceView能够在自己的线程上专注于绘图
2. SurfaceView需要我们控制当SurfaceView对象被创建，改变和销毁时如何操作
3. 需要实现线程间的同步【SurfaceViewd的渲染线程跟UI主线层<----应该就是lockcanvas()跟unlockCanvasAndPost()操作

>未确认：
1. 使用SurfaceView能够有更好的绘图性能【SurfaceView使用了更多的缓存---相较于View共有一个缓存【未进确认别乱说= =】


### **与View的异同** ###
简单点说，就是SurfaceView不支持HW acceleration，而View可以。SurfaceView使用单独的线程，而View是主线程。可见这里：[difference between surfaceview and view][difference]。  
另外也可跟GLSurfaceView进行对比,后者虽然是SurfaceView但是支持HWA：[advantage of GLSurfaceView][advantage]

Touch事件分发
---------------
>Android中跟Touch事件有关的方法有`dispatchTouchEvent()`,`onTouchEvent()`及`onInterceptTouchEvent()`。Touch事件从ViewGroup向下通过`dispatchEvent()`进行分发。在ViewGroup中如调用`onInterceptTouchEvent()`则中止事件向下分发。

Java线程
--------------
关于Java线程的几个状态，我还是有点晕= =
> 创建，就绪，执行，阻塞，结束 <---是这五个吗= =
几个关键方法自不用提了
>yield:
>sleep:
>wait:
>notify/notifyAll:

[advantage]: http://stackoverflow.com/questions/3385980/differences-and-advantages-of-surfaceview-vs-glsurfaceview-on-android
[difference]: http://stackoverflow.com/questions/1243433/android-difference-between-surfaceview-and-view