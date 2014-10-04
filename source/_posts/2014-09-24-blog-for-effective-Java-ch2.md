title: 读《Effiective Java》：创建和销毁对象
date: 2014-09-24 20:43:48
tags: [java,读书,笔记]
---
第二章 创建和销毁对象
----------
### 第1条 考虑用静态工厂代替构造器 ###
优点：
- 名称更具含义。如`BigInteger.proablePrimer()`,比调用带特定参数的构造器更能让人明白。
- 不必真的创建新对象。对于不可变类，可以预先创建对象，或者缓存创建对象，从而实现**实例受控的类**。如果原本创建的代价高，则方式可以提高性能。
- 可以返回任何子类型的对象。对于框架而言非常方便在不改动API的情况下更改实现。
- 创建参数化类型实例时，使代码更加简洁。具体看下面的例子。

缺点：
- 未提供公用类构造器的类无法子类化。
- 跟一般的静态方法难区分。可能需要查文档才能发现，因为它不像构造器那样有确定的名称。

补充说明： 
- 实例受控的类有几个有优点，确保单例，确保不可实例化。同时对于不可变类，能够保证不存在相等的实例，即`a==b => a.equals(b)`。
- 简化初始化
```java
// 简化参数化类型对象的创建-(似乎也可以称为类型推导）下面代码纯属假设
public static <K,V> HashMap<K,V> newInstance() {
	return new HashMap<K,V>();
}
// 使用
HashMap.newInstance();
```

### 第2条 多个参数优先考虑构建器 ###
对于多参数的构造器，可以使用几种方式构造对象：
- 使用*重叠构造器（telescoping constructor）*，即根据参数提供参数为[0,N]个构造器。  
> 评论：虽然可行，但是多参数时，客户端代码难写，可读性差

- 使用*JavaBean*。提供默认空构造器，再提供`set`方法。
> 评论： 1. 无法通过构造器参数来验证状态有效性，访问不合法状态的对象会崩； 2. 无法创建不可变的对象。

- Builder模式。创建builder对象来辅助构造对象。
> 客户端容易编写，易于阅读。具有名字的可选参数方便调用。同时在builder中也易于在编译时做约束检测。  
> 如果需要参数多，大多可以默认，且可能做拓展，建议一开始就使用这种模式。

补充说明：
- Java中Class存在一个`newInstance()`的工厂方法。这个方法跟Builder类似，但是这个方法总是试图调用无参数构造器，如果不存在该构造器也不会在编译时提醒，只在运行时抛异常。所以尽量避免用它。
- Builder的缺点就是，会比一般的方法还要更加冗长。

### 第3条 用私有构造器或者枚举类强化单例 ###
直接上代码，下面都是饿汉模式。
```java
// 不建议的公有域单例，不方便拓展
public class Single1 {
	public static final Single1 INSTANCE = new Single1();
	private Single1() {};
}
// 静态工厂方法，建议这种方式。拓展性好，性能不会有什么大损失（静态工厂方法自动内联化）
public class Single2 {
	private static final Single2 INSTANCE = new Single2();
	private Single2() {};
	public static final getInstance() { return INSTANCE; }
}
// 最强单例。无偿提供序列化机制，绝对防止多次实例化（不知道多进程如何?）
public enum Single3 {
	INSTANCE;
	public void method() { ... };
}
```
另：一般的单例在序列化如果不做一些专门的工作，单例的特性都会被破坏。（基本没怎么用到，不做过多补充）
> 注：在后面还会有专门的章节来讲枚举类。说到枚举类，虽然作者建议使用这个类来表示一些原本用数值常量来表示的状态，当如果是在Android中，请节制。见补充。

### 第4条 通过私有构造器强化不可实例化 ###
不做过多解释。工具类可以提供私有构造器来避免实例化。
### 第5条 避免创建不必要的对象 ###
- 构造器在每次调用的时候都会创建一个新的对象，而静态工厂则从来不要求这么做，实际也不会这么做。
- 优先使用基本类型而不是装箱类型，**小心被装箱**
- 维护自己的对象池并不是一种好的途径，除非对象真的非常重量级。对象池在占用内存的同时，也会因复杂度而影响可读性。
**补充:**
- `Calendar`的创建代价很高，能复用尽量复用，或者使用其他替代方案
- 延迟初始化**可能**提高效率。但因此而提高的复杂度却有可能降低效率-还是不要优化←

### 第6条 消除过期的对象引用 ###
- 过期引用（obsolete reference）：永远不会再被解除的引用
- 无意识的对象保持（unintentional object retention）：在支持GC的语言中内存泄露可以这么理解。
- 不要忽略一个小对象的引用，它有可能导致包括小对象所在类整个无法回收。
- 如果类自己管理内存，那么要特别注意内存泄露。比如数组，你可能划分前面部分来使用，但是垃圾收集器只会知道你占用这块内容，里面的引用都是有效的。

内存泄露的来源：
- **缓存**。你放进去了，但是忘掉它了。这个时候使用`WeakHashMap`会是个不错的选择。注意它的值的生命周期是由键的外部引用来决定的。
- **监听器和回调**。注册了，却忘记注销。

补充：
书中的这段代码，一开始我理解错了。数组的index明明总是要比`size`小1的啊= =
```java
public Object pop() {
	if (size == 0)
		throw new EmptyStackExeception();
	Object element = elements[--size];
	elements[size] = null;
	return element; 
}
```
### 第7条 避免使用finalize方法 ###
- **`finalize`通常是不可预测的，也是很危险的，一般情况下是不必要的。**

缺点：
- 不能保证该方法被及时地执行。一个对象从不可达到`finalize`被执行，所花费的时间是任意长的。不同JVM对它的实现不尽相同。
- 甚至不能保证该方法会被执行。`System.gc`,`System.runFinization`只能增加方法被执行的机会，仍然不能保证执行。
- 为类提供这个方法，会随意地延迟实例的回收过程。具体可见补充中的一篇资料。
- 如果未捕获的异常在终结过程被抛出，那该异常会被忽略，且终结过程也会终止。未被捕获的异常可能会使得对象处在不合法状态。
- 使用`finalize`会有严重的性能损失。

好处：
- 最后的防线--在忘记显式调用终止方法时，可以在`finalize`中执行它。晚回收总比不回收好。
- 清除本地对等体（native peer）。普通对象通过本地方法委托给一个本地对象，当Java对等体被回收时，它不会被回收。因此需要显示地回收掉。

补充：
- 不要依赖`finalize`来更新重要的持久状态
- 最好提供显式的终止方法。如`InputStream`等的`close`方法。另外可以结合try-catch语句，以确保终止。
- 永远记得调用`super.finalize`。因为不支持`finalizer chaining`。
- finalizer guardian - 一个用来终结外部实例的匿名成员类。（不做过多解释，个人觉得很少用到）
- 下面的补充中写了Android的doc上关于finalize的使用指南。

补充
------------
### Finalize的原理
之前在网上看到过几篇博客，讲得特别详细。简单说明下的话就是:
> 重写了finalize方法的类，VM在创建类时会检查类及父类是否重写了`finalize`方法，如果重写就会创建一个`Finalizer`对象F1引用它，F1同时也指向一个引用队列。F1还会被`Finalizer`这个类所引用，保证不会被GC。   
当重写了`finalize`对象不存在其他引用，将被回收时，它会被丢进刚才提到的`ReferenceQueue`中，等待被daemon线程中F1来调用它的`finalize`方法。而通常daemon线程优先级很低，导致回收很慢。另外调用结束后，`Finalizer`这个类会移除F1引用，这才使得重写了`finalize`方法的对象真正可被回收。

### AndroidAPI关于finalize的说明
原文可以看[参考资料][android-finalize]。Android上的说明基本跟书上提到的类似，还建议我们阅读这本书:-）  
> 不建议你重写这个方法，因为即使对象不可达，也可能要花上一顿时间才能被回收（取决于内存压力）。使用这个方法的场景也基本如上所提到的。另外还建议说，如果真的要依靠`finalize`来做点什么，建议自己用个`ReferenceQueue`，再写个线程自己处理，而不是依靠VM来清理它们。

参考资料
------------------
[Android finalize][android-finalize]
[JVM finalize实现原理][163_finalize]
[finalize方法与Java GC][another_finalize]

[android-finalize]: http://developer.android.com/reference/java/lang/Object.html#finalize()
[another_finalize]: http://106.186.122.203/?p=13
[163_finalize]: http://www.majin163.com/2014/04/26/jvm-finalize/ 