title: EventBus二三事
date: 2014-09-20 18:51:59
tags: [Android,EventBus,开源]
---
废话很多的前言
---------------
EventBus,也即事件总线。在[wiki][event_monitor]上有关于Event Monitor的一个说法：
>Event monitoring makes use of a logical bus to transport event occurrences from sources to subscribers, where event sources signal event occurrences to all event subscribers and event subscribers receive event occurrences.  
>事件监听器通过逻辑总线来将发生的事件从发生源传输到订阅者。事件发生源在事件发生的时候能够发出信号给所有订阅者，订阅者则能够接收到发生的事件。

<!--more-->

个人猜测事件总线这种说法源至于计算机内常用的几个总线结构【比如CPU总线，IO总线等】。他们有一个共同的特定就是负责传递某种 object 到指定的地方。比如数据总线负责的是CPU到RAM，地址总线负责的是RAM到RAM传递数据。

这条词条的分类是在操作系统这个目录下，这也可以理解成是在操作系统较早地使用了这种模式。  
在Java内置的包中也有一种EventBus,但它是基于接口的。Google在Guava中实现了一套自己的[EventBus][guava_eventbus],Google实现的总线库更具可读性,因为它使用Annotion标记订阅者和生产者，方法名可以自定义，更具意义。  

Android中的EventBus
----------------
在这里先说说Android内置的两种跟EventBus类似的机制。**Intent**和**BroadcastReceiver**。  
这两者都可以起到跟事件总线类似的效果。注册广播接收器和单纯发一个intent就可以唤起其他组件，提醒其他组件更新，这是非常方便的，同时也是下面提到的两个开源方案所做不到的。  
但这种机制也有不好地方，它们内部的实现都需要 IPC，虽然Android的`Binder`使得IPC简单了点~~（还是蛮复杂的吧）~~，但传递效率上会是个问题。如果自己完全不需要IPC,下面说到的两个项目会更加适合。  
在Android中似乎就只有两个Eventbus的开源项目，Square的[otto][otto]和GreenRobot的[EventBus][event_bus]（以下称GEventBus)。  
### *otto* ###
otto是square开源的一个基于Guava的EventBus的项目。跟Guava一样，otto使用Annotion作为事件的标记，`@Subscribe`即事件的处理者,`@Produce`是基本事件的生产者。举个官方给的例子：
```java
@Subscribe public void answerAvailable(AnswerAvailableEvent event) {
    // TODO: React to the event somehow!
}
@Produce public AnswerAvailableEvent produceAnswer() {
    return new AnswerAvailableEvent(this.lastAnswer);
}
// 订阅者和发布者均需要向同一个总线注册
bus.register(this);
```
使用起来就是这么简单。订阅者向总线注册，如果此时存在一个生产者，那么这个生产者的方法会被调用以产生一个初始对象来初始化订阅者。另外，在总线接收到对应的事件之后，订阅者的方法就也会被调用。  
```java
bus.post(new AnswerAvailableEvent(100));
```
post事件并不需要对象向总线注册。post事件后，如果存在事件的订阅者那么订阅者的方法就会被调用。如果不存在订阅者，那么事件会被包裹成`DeadEvent`重抛。(待做文章细说）。
默认情况下，otto只支持主线程的事件。比如下面代码所示，bus1和bus2是等价的。如果将策略设置为ANY，那么也可以在非主线程执行。只是不建议，这个库适合的场景也就只是在主线程而已，其他环境下会出现什么状况不做保证。
```java
Bus bus1 = new Bus();
Bus bus2 = new Bus(ThreadEnforcer.MAIN);
// Bus bus3 = new Bus(ThreadEnforcer.ANY);
```
#### **小问题** ####
要想在基类中注册事件要花点心思。（不能通过this来向bus注册事件，因为在子类中的方法调用super到父类方法时，this指的永远是子类）。要想在基类注册事件，基本上只能写一个成员变量，在变量内部注册方法。逻辑会略奇怪。另外，可能需要自己实现一个[BusProvider][busprovider]

### *GEventBus* ###
greenrobot的另一个比较出名的开源项目（另外一个是greenDAO）。同样是EventBus，但这个项目不是基于Annotion的。因为Android的Runtime Annotion处理起来效率实在是不高，尤其是在4.0之前。~~在4.0之后也不见得效率有多大提高~~。  
GEventBus使用的是字符串匹配，默认订阅者会调用带`onEvent`或者说是EventBus类里面的`DEFAULT_METHOD_NAME`作为**前缀**的方法。只有仔细跟踪代码之后，你才会发现它的几种线程模型是这么用的:
```java
public void onEvent(SimpleEvent event) {};
public void onEventBackgroundThread(SimpleEvent event) {};
public void onEventMainThread(SimpleEvent event) {};
public void onEventAysnc(SimpleEvent event) {};
```
其他操作跟otto类似。只是GEventBus不支持生产者方法。(题外话：我是看了otto的demo之后才知道GEventBus的一般用法= =）

#### **问题** ####

1. 跟otto一样，如果用继承可能会存在问题。但具体的问题大概恰好相反，如果一个父类向总线注册了事件，在otto里面是只有子类注册了事件，但在GEventBus中会是子类和父类都注册了。如果父类中注册了总线，那么子类中**必须**实现一个`onEvent*`方法，否则程序就会崩掉。解决方案，也算有吧，自己实现了一个蛮挫的暂时性解决方案 [issus98][issues98]
2. 默认方法是只能带一个参数的，如果传入多个参数，那么GEventBus会完全忽略它，而且也不会报任何错误。这是个小细节，注意看文档的话大概也知道只能用一个参数，但是相较于otto会对参数不符合主动抛出一个`RuntimeException`,我始终觉得GEventBus有问题。

个人看法
--------------------
> 注意以下会有很多个人的喜好导致的吐槽=_=

个人觉得最纯正的是otto,即使GreenRobot的EventBus称它的效率比前者高出许多（有benchmark，所以算是事实）。但是一个开源项目，看的并不只是效率问题吧。  
在知乎上看到过一个问题：[一个开源软件为何成功][zhihu_nice_open_src]。一个好的开源项目也是同理，其中有个答案提到的几点我非常赞同。
1. **对新手友好。**  
这点在EventBus上我完全看不到。没有Demo，文档只是关于项目的一个简单介绍，还有吹嘘他们比起otto效率如何如何地高,功能多强大。  
另外还有一点，EventBus的文档极具误导性。对于TODO的feature，它竟然摆在功能介绍的模块里，**即使它前面加上了没有加粗的TODO**。在Trinea的开源集合里面就把那些TODO的内容加到了EventBus的功能里面。
当初在看到它的效率如此地高，就决定选它了。可是没有demo，完全不知道怎么入手。
2. **及时接收反馈。**  
EventBus上现在还有还有2012年的issues未关闭，而且很多都没有项目的人出来解答。不像otto，只要不是太弱智，基本上每个issues都会有反馈。

在发现没有Demo之后，我研究过GEventBus的代码。逻辑是搞懂了，但也发觉了GEventBus的代码是如此地不友好，没有关键的注释，名字又表意不清。最最无法容忍的是，它将一堆函数标记为`@depreated`,即使现在只能用它们来做。这个Annotion是来这么用的吗？提供了新的替代接口再标记为废弃如何？或者你干脆设置成private好么。  
看otto的代码是享受，虽然我只是稍微看了一下，没有深入去研究。Square似乎还为otto开发了IntellJ的plugin，可以快速地浏览所有的订阅者和生产者,这点也是GEventBus所不及的吧。  
就个人喜好而言，果然还是选择otto，用annotion写起来还是比较顺心的。不像GEventBus，竟然要用那么长的名字才能作为事件回调。**输入越多，越容易出错**，这点无论是在用户体验还是敲代码的时候都是同理。
个人拙见，欢迎交流>_<  22:09


参考资料：
----------------------
[Android Annotion的bug][android_annotion_bug]
[总线-百度百科][bus_in_baike]
[Guava EventBus][guava_eventbus]
[EventBus][event_bus]
[otto][otto]

[issues98]: https://github.com/greenrobot/EventBus/issues/98
[android_annotion_bug]: https://code.google.com/p/android/issues/detail?id=7811
[busprovider]: https://gist.github.com/JakeWharton/3057437
[otto]: https://github.com/square/otto
[event_bus]: https://github.com/greenrobot/EventBus
[guava_eventbus]: https://code.google.com/p/guava-libraries/wiki/EventBusExplained
[event_monitor]: http://en.wikipedia.org/wiki/Event_monitoring
[zhihu_nice_open_src]: http://www.zhihu.com/question/25393186/answer/30644774
[bus_in_baike]: http://baike.baidu.com/link?url=yc55M8ZPNh23sDc5q0meSq2iTHZ3zpgIsuPMSIVn1mnVvdxQKVOjaIYShjs9T7M9