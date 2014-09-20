title: Android性能优化
tags: [android,优化]
---
常见的性能优化点：
对象池机制
---------------
`Message`和`PendingIntent`都提供了类似于`obtain`的方法，使用这种机制可以减少创建对象的次数。

适当的数据结构
-----------------
Android中内置的`SparseArray`系列数据结构能够有效地代替Java中使用`Integer`作为`Map`的key的泛型。   

而提到为什么不用`Integer`,则是因为使用它增加了时间和空间上的成本。自动装箱和拆箱虽然减少了编程成本（不用显式cast），但是在数据量大的情况下经常就是性能杀手。

Parcelable性能更加
-------------
Parcelable是Android专门针对在Binder传递数据进行优化的一个接口，使用它来代替Java的Serialable接口能够有效地提高传递效率。

AIDL中使用恰当的方向标识
-----------------------
方向标识（Direction Tag）也就是 in, out, inout这三种。默认为元类型都添加了in标识。对于自己创建的对象，使用恰当的tag能够提高marshalling时的效率。