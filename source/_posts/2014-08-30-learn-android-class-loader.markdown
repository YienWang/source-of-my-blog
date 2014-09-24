---
layout: post
title: "learn android class loader"
date: 2014-08-30 13:09:11
comments: true
tags: Android
---

昨晚看到有人在讲这个话题，突然间也就想要研究一下了。Android的类加载机制跟Java的有什么异同。

<!--more-->
#### 关于Java的类加载机制
##### Java的类加载器类型：
1. 启动类加载器 Bootstrap ClassLoader  
	这个类无法被程序直接引用。如果需要由它加载，在自定义类加载器的时候只需要把null作为双亲加载器的引用传入即可。  
	默认在lib文件夹下
2. 扩展类加载器 Extension ClassLoader  
	默认在在lib\ext文件夹下
3. 应用程序类加载器 Application ClassLoader  
	加载在ClassPath路径上所指定的类库


Java使用了双亲委派模型来进行类加载。  
当一个类加载器收到了类加载的请求，它首先会将请求委托给父类加载器去完成，层层递归。因而所有的类都会被启动类加载器查找过，如果父加载器找不到才交给子加载去加载。

##### Java类加载的过程：
1. 加载  
	读取二进制流，在内存中生成一个Class对象
2. 验证  
	连接阶段的第一步。确保Class文件的字节流符合当前虚拟机的要求
3. 准备  
	为类变量分配内存并设置类变量初始值（默认都为0值，除非该字段为final）--也就是说非final的static变量初始值都是0值，在初始化时才被初始化为给定的初始值
4. 解析  
	虚拟机将常量池中的符号引用替换为直接引用的过程。连接的最后一阶段。
5. 初始化  
	最后一步。按照代码初始化类变量及其他资源的过程。


###### 进行类初始化的时机：
1. 使用new实例化对象时，读取或设置一个类的非final的静态字段时,调用一个静态方法时
2. 使用反射方式调用时，如果类未初始化过，那么也会调用初始化。
3. 初始化一个类时，如果父类未初始化，那么就先初始化父类
4. VM启动时，会先启动包含main的主类。

#### 关于Android的类加载机制
在网上始终找不到好的资源来介绍这个内容。官方对类加载器的介绍也只是在 `DexClassLoader`上，其目的是提供一种动态加载的机制。这也就是现在插件式应用所用的技术了。  
虽然没什么资料，介绍性的东西还是需要的。  
先看看官方API上的说明：
>Loads classes and resources from a repository. One or more class loaders are installed at runtime. These are consulted whenever the runtime system needs a specific class that is not yet available in-memory. Typically, class loaders are grouped into a tree where child class loaders delegate all requests to parent class loaders. Only if the parent class loader cannot satisfy the request, the child class loader itself tries to handle it.

基本作用跟Java的类似，在双亲委托机制上也是跟Java一样的。子加载器会先把所有的请求委托给双亲加载器。只有双亲无法加载的情况下才会将类交给子加载器加载。  
在MainActivity中尝试把类加载器的层次show出来，结果是这样子的：
```java
D/Loader(18811): dalvik.system.PathClassLoader[dexPath=/mnt/asec/com.example.parcelablepro-1/pkg.apk,libraryPath=/mnt/asec/com.example.parcelablepro-1/lib]
D/Loader(18811): java.lang.BootClassLoader@41af8ed0
```
可以看到，这里的类加载器只有两层----顶层类是BootClassLoader,在ClassLoader的代码中可以找到这个包可见的内部类：
```java
/**
 * Provides an explicit representation of the boot class loader. It sits at the
 * head of the class loader chain and delegates requests to the VM's internal
 * class loading mechanism.
 */
class BootClassLoader extends ClassLoader {
	...
}
```
而这个类，内部是使用了` VMClassLoader.loadClass(name, false)`来加载。实际上也就相当于对BootstrapClassLoader的一个封装。再往下挖：
```java
//VMClassLoader.class
/**
 * Load class with bootstrap class loader.
 */
 native static Class loadClass(String name, boolean resolve) throws ClassNotFoundException;
// resolve参数用于指明是否对class进行验证（链接），android默认是不进行验证。从而提高加载速度
//stackoverflow: The resolve parameter controls whether the class that's loaded is linked or not.linked见Java类加载过程
```
关于Android的类加载器，在ClassLoader有一个静态方法`ClassLoader.getSystemClassLoader()` 有如下说明：
>Returns the system class loader. This is the parent for new ClassLoader instances and is typically the class loader used to start the application.

返回系统加载类。也是这个加载这个类加载器的父类加载器，通常也是用于启动这个应用的类加载器。Log出来的结果如下：
```java
D/Loader  (19792): dalvik.system.PathClassLoader[dexPath=.,libraryPath=null]
```
由此可以看出，一个应用的类加载器通常都是由PathClassLoader进行加载的。而加载这个PathClassLoader，通常也就是BootClassLoader。应用层都是PathClassLoader进行加载，而BootstrapClassLoader则是用于加载框架层的类加载器。
##### 关于插件式开发
最常见的应该就是 `DexClassLoader` 这个类了吧。插件式的应用，基本上都是使用这个类加载存放在其他地方的类的。
```java
	String libPath = WHERE_IS_YOUR_DEX;
	final File tmpDir = getDir("outdex", Mode.PRIVATE);
	final DexClassLoader classloader = new DexClassLoader(libPath, tmpDir.getAbsolutePath(), null, this.getClass().getClassLoader());
	//public DexClassLoader (String dexPath, String optimizedDirectory, String libraryPath, ClassLoader parent)
	// dex所在地址;优化后的代码应该放置的地点;对应的lib地址;可以为null;通常是直接this.getClass().getClassLoader())
    final Class<Object> classToLoad = (Class<Object>) classloader.loadClass("org.shlublu.android.sandbox.MyClass");

    final Object myInstance  = classToLoad.newInstance();
    final Method doSomething = classToLoad.getMethod("doSomething");

    doSomething.invoke(myInstance);
```

[bootclassloader]: https://android.googlesource.com/toolchain/gcc/+/refs/heads/master/gcc-4.3.1/libjava/gnu/gcj/runtime/BootClassLoader.java