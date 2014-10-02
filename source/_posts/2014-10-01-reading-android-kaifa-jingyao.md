title: 读《Android开发精要》
date: 2014-10-01 10:03:34
tags: [android,笔记,读书,问题]
---
前言
----------
之前是翻过了一遍，只是没做总结的话似乎没什么印象。接下来就简要地总结一下书上提到的内容【估计也不简要-我做笔记总是会太注重细节】。内容会按照章节来做总结，不再细分小点。存在拓展性内容的话会重点描述，如果是官方的资料，那这里只会给出相应的链接。

笔记
---------------
### 第1章 Android的系统架构
#### Android系统概况
组成部分是**应用层，框架层，运行时，核心类库，硬件抽象层（HAL）,Linux内核**。  
（在网上搜到的图基本上都忽略了硬件抽象层。这一层似乎是为了统一接口，向上屏蔽底层硬件的细节而存在。该层跟Linux间似乎也有一个bionic隔着。bionic是google实现的c系统库，为了防止GPL向上传染）
![android][1]
1. **应用层**
这一层基本用Java开发，也可以通过JNI调用C/C++的类库。在API9之后Android提供了`NativeActivity`来为用C/C++开发应用提供便利。但Google并未提供控件在核心类库的实现，所以如果使用C/C++开发，自己将无法使用控件，只能使用GL系列自己绘图。
2. **框架层**
最核心的部分，有多个系统服务组成。这些服务运行在系统核心进程，每个服务都是一个独立的线程。这些服务与外界通过binder进行IPC和RPC。  
应用由多个组件组成，组件间的通讯是通过框架层的系统服务进行，集中的调度和传递信息，类似于外观模式(facade pattern)。
3. **运行时**
由Java的核心类库和Java虚拟机[Dalvik][2]共同组成（曾经是，从5.0开始都是ART）
Dalvik使用采取了**基于寄存器**的虚拟机架构设计：基于寄存器的的虚拟机对硬件门槛更高，编译出的应用可能占用更多空间，但执行效率更高。  
Dalvik没有采用字节码，而是在编译时将所有.class文件转换成.dex文件,使用的指令集为OpCodes。
4. **核心类库**
由一系列的二进制动态库共同组成，按需要而被加载到系统服务中。
5. **HAL和Linux Kernel**
HAL是Android为厂商提供的一套接口标准，为框架层提供接口。

#### 架构特征和设计思想
Android的应用是高度组件化的，一个功能可能由多个组件共同完成，不会有明显的进程边界和应用边界。 

### 第2章 源码获取与编译-略
### 第3章 Android组件模型解析
#### 基于Mashup的应用设计
基于组件的应用设计模式：每个应用由一系列的组件搭建而成，组件通过配置文件描述功能。
在Mashup的概念下描述应用，有三个基本元素：组件，连接和配置。
1. **组件**：有特定功能和接口规范的实现单元。
2. **连接**：组件与组件间的通信信道。Android中提供的机制有Intent和Uri。
3. **配置**：描述组件的功能和实现特征的信息。

#### Activity解析
- Android系统是个多任务的操作系统，对于每个任务都存在一个Activity栈与之对应。
- Activity的实例则存在栈中，处在栈顶的组件即可见可交互的前台组件。
- Activity继承自Context。ContextImpl和ContextWrapper各自实现了Context。但前者使用了ContextImpl的实例来完成功能，因而使得两者可以独立变化，彼此独立。
> **界面开发小tips**
> 1. 能用抽象控件尽量不要具体。在代码层使用View，而资源文件使用具体类型。【个人不赞同，如果需要频繁更改界面控件那反而应该反思，而不是写出这种代码
> 2. 能用资源文件描述就不要用代码控制。【赞同，比如按下状态完全可以用xml中`selector`来实现。
> 3. 降低界面复杂度，能用XML就用XML。【同上
> 4. 不要在控件回调事件中放具体的代码。【提前告知了重构的规则，防止耦合。只是有时并没必要

#### Service解析
- Service默认是不会运行在独立的进程或线程中，而是运行在应用进程的主线程中。
- Service既可以像Activity一样作为调度者，也可以作为功能提供者。
- Service提供两者使用方式，`startService`和`bindService`。这两者不同的地方在于，前者会导致`onStartCommand`被调用，但是后者是`onBind`被调用。通常前者还需要调用`stopSelf`来终止自己。  
一般如果只是`startService`，建议使用IntentService重写`onHandleIntent`来代替。IntentService自动创建后台线程来提供服务。
~~书上在后面扯到了AIDL，这里不再对其进行说明~~
> 小tips：
> 通过`getSystemService`拿到通常是该服务一个代理对象，这些对象与真正的服务线程建立连接，通过IPC调用来实现对应的方法。

#### BroadCast Receiver解析
通常只会调用`onReceive`,生命周期也仅限于此。  
Receiver接受两种注册方式，动态和静态两种方式【不想使用热插拔和冷插拔的说法】。
1. 静态方式。在配置文件中声明receiver，可以设置`Intent-filter`，也可以不设置。
2. 动态方式。`registerReceiver()`和`unregisterReceiver()`。通常电量变化和时间改变的通知都是通过这种方式来注册的【因为这些事件通常是sticky（粘性）的，时常发生，使用静态方式需要频繁构造和销毁Receiver。】

发送广播事件的方式也有两种。`sendBroadcast()`和`sendOrderedBroadcast()`，前者是普通广播事件，后者是有序广播。有序广播按照优先级次序依次发送，高优先级的Receiver可以调用`abortReceiver()`中止事件继续传播。另外，广播中也可以通过`setResult`和`getResultData`传递特定的数据。

> 个人tips:
> 发送广播要特别注意安全的问题的。因为如果在配置文件设置了action，那么其他应用都可能调用到。为了保证广播只在本地进程被调用，可以使用[LocalBroadcastManager][5]

#### ContentProvider解析
- 负责为应用提供数据访问接口的组件，建议在应用共享数据时使用。它通过URI定位，使用SQL语句操作内容。  
- 在主线程构造和执行操作，如果需要异步，可以使用`AsyncQueryHandler`。  
- 操作数据时，通常使用ContentResolver。该类实例内缓存了ContentProvider的代理对象，从而可以通过“定位-执行”的这样的操作实现功能。所有的操作都是在消息队列中串行执行的，无法并发操作。   
- ContentProvider在配置时可以设置`multprocess`,该项代表是否在另外一进程创建一实例。这样的好处在于减少IPC操作【不过数据的同步如何保证？复杂度会大大提高，所以不甚建议】

### 第4章 Android的Intent机制 略
【个人觉得读过，了解一些常用的设置即可。也有可能是我现在没怎么接触到这样的需求，现在不做过多解读。经验多了，再来看这段话也许会觉得自己很无知？】
```java
Intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
Intent.setType("image/*");
...
IntentFilter.setPriority(1000); // -1000~+1000
```

### 第5章 组件生命周期解析
#### 应用进程模型
**定义：**Android应用运行时，应用进程的分配和调度方式，以及应用组件和进程间的关系。
- 默认情况下，应用进程名称与包名相同。
- 默认配置中，所以组件均在主线程。ApplicationContext在第一个组件在第一个组件加载前被初始化，在最后一个组件结束时被销毁。这个上下文对象为所有组件提供全局的功能和数据支持。
- 在manifest中通过`android:process`可以配置进程。如果名称前有冒号，说明是私有进程，仅有该应用的组件能够置入。如果是以小写字符开头则是共享进程，其他线程可访问。【可以参考这里`android:process`的[用法][6]  
**实际开发中，通常需要将在逻辑上一起运行的组件配置到统一进程中。如一个共享进程中的Service，一个Receiver就必须跟它同一个进程。否则很有可能发生onReceiver时创建了一个进程，再去调用另外一个进程，最后又结束了自己的进程的情况。**【这个场景真心没想过，不过确实有可能发生

#### 应用进程托管
**定义：**Android的应用进程的构造和销毁均有系统统一调度。
- 为何?
	1. 简化开发模型。组件间的跨进程通信的细节不对上层公布。
	2. 方便全局调度。优化进程对系统资源的使用。
- 进程优先级：前台，可视，服务，后台，空进程
   如何区分？区分原则比较啰嗦，前台进程会涉及到很多组件的生命周期。[具体的区分原则][7]
- 进程回收。【跟Intent-Filter匹配类似的内容，不想赘述，记忆性的内容。
（小疑问，关于`onSaveInstanceState()`到底做了什么，仍然抱有疑问。真的是作者说的存到磁盘？）
- 终止进程。如果进程发生了意外，无法继续执行而成为异常进程。此时Android会终止这个进程，但是会先保留它的stack，并重新创建进程和恢复它的stack，除了最顶层的Activity。
- ANR：此时的终止是不会给你复活的← ←
	1. 对用户的操作（键盘或触屏事件）超过5秒未响应。
	2. 广播接收器的onReceiver超过10秒。

#### 组件的生命周期
- 从构造运行开始到被销毁的时段内，组件的状态变化。（可以思考一下为什么Android要提供这种状态变化）
- 各种组件的具体生命周期。【Activity除了状态缓存，基本没有什么，接下来是一些补充。
- 关于Service的`onStartCommand`的返回值：
	- `START_STICKY`，即使被强行回收了，有空闲内存时仍会再调用它，除非服务自己调用`stopSelf()`；
	- `START_NOT_STICKY`，系统不管服务是否完成，无条件终结后不再负责复活。
	- `START_REDELIVER_INTENT`，保证Service能够完整地处理每个Intent对象。如果处理到一半被终结，则再调用，再投递同一个Intent。

#### Task和Activity Stack


问题
---------------
写完这篇博客之后尚未解决的疑问，估计要研究一下
1. `onSaveInstanceState()`到底会做什么？作者一开始说存到磁盘，后又说是使用bundle形式保存，明显存在矛盾。另外，关于`onSaveInstance()`与`onRestoreInstanceState()`的调用时机也不明确。作者说如果是用户按下HOME键主动退出是不会调用的，果真如此吗？
2. SQlite是表级别的锁，那么支持并发吗?

碎碎念
----------------
国庆的第一天，中午饭就不想吃了。睡了一个中午，也不饿。我对假期总有这有的感觉吗 10.1 15:15

[1]: http://www.anddev.org/images/android/system_architecture.png
[2]: https://source.android.com/devices/tech/dalvik/index.html
[3]: http://stackoverflow.com/questions/3320534/android-application-architecture-what-is-the-suggested-model
[4]: http://www.cubrid.org/blog/dev-platform/binder-communication-mechanism-of-android-processes/
[5]: http://developer.android.com/reference/android/support/v4/content/LocalBroadcastManager.html
[6]: http://stackoverflow.com/questions/7142921/usage-of-androidprocess
[7]: http://developer.android.com/guide/components/processes-and-threads.html#Lifecycle