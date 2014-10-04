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
- 默认配置中，所以组件均在主线程。ApplicationContext在第一个组件加载前被初始化，在最后一个组件结束时被销毁。这个上下文对象为所有组件提供全局的功能和数据支持。
- 在manifest中通过`android:process`可以配置进程。如果名称前有冒号，说明是私有进程，仅有该应用的组件能够置入。如果是以小写字符开头则是共享进程，其他线程可访问。【可以参考这里`android:process`的[用法][6]  
**实际开发中，通常需要将在逻辑上一起运行的组件配置到统一进程中。如一个共享进程中运行着Service，要调用Service的Receiver就必须跟它同一个进程。否则很有可能发生onReceiver时创建了进程，再去调用另外一个进程中Service，最后又结束了自己所处进程的情况。**【这个场景真心没想过，不过在广播时确实有可能发生

#### 应用进程托管
**定义：**Android的应用进程的构造和销毁均由系统统一调度。
- 为何?	
	1. 简化开发模型。组件间的跨进程通信的细节不对上层公布；
	2. 方便全局调度。优化进程对系统资源的使用。
- 进程优先级：前台，可视，服务，后台，空进程
   如何区分？区分原则比较啰嗦，前台进程会涉及到很多组件的生命周期。[具体的区分原则][7]
- 进程回收。【跟Intent-Filter匹配类似的内容，不想赘述，记忆性的内容。
（小疑问，关于`onSaveInstanceState()`到底做了什么，仍然抱有疑问。真的是作者说的存到磁盘？）
- 终止进程。如果进程发生了意外，无法继续执行而成为异常进程。此时Android会终止这个进程，但是会先保留它的task，并重新创建进程和恢复它的task，除了最顶层的Activity。
- ANR：此时的终止是不会给你复活的← ←
	1. 对用户的操作（键盘或触屏事件）超过5秒未响应。
	2. 广播接收器的onReceiver执行超过10秒。

#### 组件的生命周期
- 从构造运行开始到被销毁的时段内，组件的状态变化。（可以思考一下为什么Android要提供这种状态变化）
- 各种组件的具体生命周期。【Activity除了状态缓存，基本没有什么，接下来是一些补充。
- 关于Service的`onStartCommand`的返回值：
	- `START_STICKY`，即使被强行回收了，有空闲内存时仍会再调用它，除非服务自己调用`stopSelf()`；
	- `START_NOT_STICKY`，系统不管服务是否完成，无条件终结后不再负责复活。
	- `START_REDELIVER_INTENT`，保证Service能够完整地处理每个Intent对象。如果处理到一半被终结，则再调用，再投递同一个Intent。

#### Task和Activity Stack
- 在Stack中，不处在相邻位置的Activity无法相互调用。
- 启动模式有4种。划分规则是1.是否复用对象；2.是否只作为栈的根组件而存在。singleTask和singleInstance适用于消耗内存较多的单实例Activity，如浏览器和音乐播放器。
- Task Affinity（Task亲和度）：组件期望和哪些组件分配到同一个Task中。常在`android:taskAffinity`配置同一名称的Task。另外还需要另外配置`android:allowTaskReparenting`或者设置Intent的Flags为`FLAG_ACTIVITY_NEW_TASK`。关于后面那种设置，一般是要将组件作为Root Activity，但如果设置了亲和度，则会先去寻找具有同名的Task。

### 第6章 组件间的数据传输
#### 使用Intent对象传输数据
- 在`startActivity`和`startActivityForResult`等启动组件的时候使用。【知道使用方式即可。result中有两个默认选项`RESULT_OK`,`RESULT_CANCEL`。`RESULT_CANCEL`的情况比较少见，但也可能会遇到，此时的`startActivityForResult`是调用后马上返回的，所以比较好判断。】
- **优点：**开发便捷，跨进程和跨应用。
- **缺点：**传输开销大（至少两次序列化和反序列化）,不适合在多个组件间共享数据（代码耦合和实现成本高）
- Intent适合于两个组件间点对点传输。如果传输的数据较大，建议存在磁盘，再通过uri传输【小米的Intent中不能放较大bitmap，不然就会跪= =】

#### 使用文件进行数据共享
- `Context.openFileOutput()`可以创建或打开私有的文件。
- **优点：**方便多个组件对象间的数据共享和传输
- **缺点：**读写开销大，开发成本较高，若在SD卡上安全问题严重
- 有持久化需求的小数据共享【我更加建议用Preference

#### 使用Application全局共享
- 继承Application并在manifest中配置
- **优点：**效率高，可实现不需要持久化的数据共享
- **缺点：**不支持跨进程和线程，可能占用更多的空间

#### 利用组件共享数据
- 使用ContentProvider.**优点：**更安全，快捷，适用于关系型数据
- 使用Bound Service简单共享数据

### 第7章 Android控件解析
#### Android的控件框架
- 控件均继承自View，包括ViewGroup。
- 控件最终形成一棵控件树。上层父控件负责下层子控件的排列和绘制，并将交互事件自上而上传递到子控件。每棵控件树都有一个ViewParent对象与其根控件绑定，这个对象是整个树交互事件的控制枢纽。控件树中每个控件都包含一个指向ViewParent的引用，当子控件的焦点尺寸改变时会通知该ViewParent对象由该对象统一向下分发。
- 交互事件传递。当父控件收到交互事件后，会先判定事件的目标控件对象，如果自己需要则处理（可能拦截），否则向下分发，直到事件被处理或被忽略。        
- 控件的测量和绘制。从上到下一次调用`measure()`测量和`layout()`定位，最后调用`onDraw()`绘图。【`onMeasure()`和`onLayout()`可被重写，但上面两个方法是final的】

#### Android的窗口机制
- Android将用户的操作转变成事件传递到交互界面上是通过窗口机制来实现的，即基于窗口注册的实现模式。
-  WindowManger统一管理所有的ViewParent（实现类是ViewRoot）。在创建完界面后，ViewRoot会向WindowManger注册，并建立双向通信的连接。当产生事件时，系统底层框架将事件转换并传递到窗口管理服务中，窗口管理服务解析操作并转换为交互事件。接着再定位当前与用户交互的窗口，将交互事件转发给对应的ViewRoot。
-  WindowManger会为每个窗口分配一个窗口层次z-order。越晚添加的窗口，窗口层次越高。另外，系统的交互模块会具有更高的基础窗口层次。
-  每个Activity都会有一个Window对象，每个对象都会负责构造和管理一棵控件树，并为该控件树构造对应的ViewRoot对象与窗口管理服务的双向通信。
-  可以通过`requestWindowFeature()`设置窗口的样式。注意这个方法需要在`setContentView`前调用。如果是`setFeatureInts`则需要在之后调用。
-  每个Dialog也包含一个窗口对象【**在使用Builder时传入的context不能为Application，因为Application不能调用窗口服务，谨记。**
- PopupWindow，弹出窗口，该控件不包含Window对象，它自身自行管理控件树与窗口服务通信。PopupWindow依赖于AnchorView(锚点控件）的窗口服务。而AnchorView的窗口服务通常是异步建立的【在setContentView时不一定完成】，所以需要将PopupWindow的初始化操作post进AnchorView的消息队列中，等待其连接建立后再初始化。

#### Android基本控件 【简略
- TextView有一个很重要的辅助类来实现富文本：SpannableString【图文混排怎么做？
- SurfaceView具有独立的窗口，而不需要受父控件控制。但同时它也需要处理自己的布局及大小测量。

#### 自定义控件和自绘控件 【略
【简单介绍了一下自定义控件的类型还有SurfaceView系列，其他没什么了

### 第8章 Android应用资源
#### 构成
- 在raw文件夹中的文件需要通过`Resource.openRawresource()`打开。而在assert目录的文件需要使用AssertManager来读取。
- 可以在`android:configChanges`中声明不关注的设备配置项，当这些配置项目改变时不需要销毁和重新构造组件，而直接调用`Activity.onConfigurationChanged()`来通知配置更改事件。常用的配置的有`orientation`和`keyboardHidden`。

#### 调用
- Android的资源文件的处理：预编译，编译及打包。【注意这个过程在4.0后已经改变
	- 预编译：使用aapt对资源文件进行解析，生成R和App._ap文件
	- 编译：将R文件与其他文件一起编译成.class，再用dx打包成.dex
	- 打包：使用apkbuilder将dex，资源文件等打包成apk

### 第9章 数据存储
- 结构目录。应用的安装包会被放在/data/app目录下；在安装时会将dex文件解析成odex文件后存储在dalvik-cache目录下；应用产生的数据会被放在/data/data/目录下；

> 小tips：
> 在2.2后，可以通过在manifest中配置`android:installLocation="preferExternal"`将应用安装到外部存储中，但/data/data文件仍然是在内部存储中【安全问题

#### 使用数据库。
- Android在Java层使用CursorWindow机制来控制对数据库的读取。并非一次性地取出所有资料，而是构造一个数据窗口来动态映射部分数据行，将这些数据行从数据库中读取出来并缓存在上层对象中。访问时如果命中，则直接返回，否则通过JNI再去调取数据。
- 为了提高性能，可以通过SQLiteStatement预编译SQL语句。
- 通过同一个SqliteDatabase对象访问数据库时线程安全的，因为Android会对数据库进行加锁保护。如果在使用时，出现了指向同一个数据库的多个对象同时在多个线程中被使用，就会抛出`database is locked`的异常。
- 在实践中，需要保证同时访问数据库的SqliteDatabase对象仅有一个。
- 【关于SQLite的使用待做文章细说。

### 第10章 网络通信
#### Web通信
- 使用HTTPClient。步骤:1 实例化HttpClient；2 实例化 HttpUriRequest；3 接收处理HttpResponse
- 默认情况下DefaultHttpClient（HttpClient的一个默认实现）是线程非安全的，不能同时在多个线程中同时调用DefaultHttpClient对象的方法。如果有线程安全和超时控制的要求，可以使用AndroidHttpClient。
- P2P，NFC及WIFI等略【只是简单介绍，真需要使用的时候也可以很快理解

### 第11章 地理信息服务 略
【如果有需要的话，自己查去

### 第12章 多媒体处理
#### 图像的处理
- 永远记得调用`Bitmap.recycle()`。【为了3.0之前的手机能够哦正常释放Bitmap的Native peer
- Nine-Patch：图像左，上的区域控制图像区域拉伸，右，下的区域控制文字出现的范围。
- Bitmap.Options这个类中，in\*系列代表图像读取的输入参数，out\*系列代表图片的基本信息，如果长宽，类型等。

#### 音频 略
#### 相机的使用
- 可以通过`Camera.getCameraInfo()`来获取相机的基本信息。
- 在Android中，任何一个相机的资源都是独占的，永远只有一个程序能够控制相机。所以通常会在onResume时获取，onPause时释放。
- 相机的预览通常使用SurfaceView。为了使预览过程足够清晰，Android使用YCrCb格式存储预览数据。如需转码，还请自己来。

### 第13章 其他模块
#### AppWidget【有需要再复习

勘误
--------------
- P151，事件是在控件树从上往下，而非相反
- P241，通过`getWriteableDatabase()`和`getReadableDatabase()`获取的数据库均可以读写，而不是后者仅限于读。作者理解有误
- P285，调用`Bitmap.recycle()`之后并不是马上释放了内存。只是告知这个Bitmap可以回收而已。在3.0之前这个操作的意义还在于释放native peer（这也可能是作者说马上释放了内存的原因）。而在3.0之后，Bitmap是在VM的堆中分配的，可以不调用`recycle()`，而只要将Bitmap的引用置为NULL即可。吗
- 出现多次的错误：Application的`onTerminal()`事件只会在AVD中触发，真实设备是不会发生该事件的。作者把这个事件理解成跟`finalize()`类似的概念了。


问题
---------------
写完这篇博客之后尚未解决的疑问，估计要研究一下
1. `onSaveInstanceState()`到底会做什么？作者一开始说存到磁盘，后又说是使用bundle形式保存，明显存在矛盾。另外，关于`onSaveInstance()`与`onRestoreInstanceState()`的调用时机也不明确。作者说如果是用户按下HOME键主动退出是不会调用的，果真如此吗？
2. SQlite是表级别的锁，那么支持并发吗?Locked状态是怎么回事？
3. 图文混排如何实现？

碎碎念
----------------
-  再次翻完了并做了笔记，比预料的多花了不少时间。如果只是记录一些自己不是太了解的内容，这本书上能写的应该不多吧。笔记应该是自己所不熟悉的，还有就是一些简洁的总结性内容。下次是不是可以只是选择性地只记录一些概要性的内容,对于已知的不做过度描述。  10.4 21:56
- 国庆的第一天，中午饭就不想吃了。睡了一个中午，也不饿。我对假期总有这样的感觉吗 10.1 15:15

[1]: http://www.anddev.org/images/android/system_architecture.png
[2]: https://source.android.com/devices/tech/dalvik/index.html
[3]: http://stackoverflow.com/questions/3320534/android-application-architecture-what-is-the-suggested-model
[4]: http://www.cubrid.org/blog/dev-platform/binder-communication-mechanism-of-android-processes/
[5]: http://developer.android.com/reference/android/support/v4/content/LocalBroadcastManager.html
[6]: http://stackoverflow.com/questions/7142921/usage-of-androidprocess
[7]: http://developer.android.com/guide/components/processes-and-threads.html#Lifecycle