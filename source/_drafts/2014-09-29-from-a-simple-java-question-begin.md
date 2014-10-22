title: 从一个简单的问题开始讨论Java继承初始化
date: 2014-09-29 08:59:37
tags: [java,初始化,继承]
---
前言
--------------
在OSC上看到一个**简单的**[问题][1]，一开始觉得蛮简单的，但后来仔细想想，似乎蛮难搞清楚的。大概以后都不能小看那些看起来简单的问题啊~好了，不说废话=。=

正文
--------------
问题的具体内容如下：
> 下面的代码中的输出为什么是这个，尤其是第二个输入。还有`this`的强制类型转换为什么可以成功?

```java
class ABC extends AB {
	public int a = 100;
	public ABC() {
		System.out.println(a);
		a = 200;
	}
	static {
		System.out.println("static A");
	}
	public static void main(String[] args) {
		new ABC();
	}
}
class AB {
	public AB() {
		System.out.println(((ABC) this).a);
	}
}
```
输出会是
```java
static A
0
100
```
第一个和第三个都可以理解，但是为什么第二个会是0呢？而且更加神奇的是，为什么这段代码可以编译通过，为什么不会抛`ClassCastException`？

### 类加载的顺序
之前小总结过java的类加载机制，基本算是弄清楚了。但是如果再考虑继承，java的类加载过程又会是怎样的呢？

### this
this在这里指向的是什么，这里值得讨论一下。先简单介绍一下this。
官方的说法是这样子的：
> Within an instance method or a constructor, this is a reference to the current object — the object whose method or constructor is being called. 
> 在一个实例方法或构造器中，this是正在调用这个方法或构造器的对象的引用。

按照官方的说法，似乎在父类中调用的`this`应该是父类才对，但是实际上this指向的是子类。而且如果把上面的`((ABC)this).a`中的强制类型转换去掉，这段代码在编译时即报错：
```java
Unresolved compilation problem: a cannot be resolved or is not a field
```
this的引用是在运行时才确定的，如果使用了继承，在编译器在编译的时候传入的this是指向当前构造器的对象，不存在子类的域，因而如果不强制类型转换为运行时的子类编译会不通过。

在stackoverflow上看到类似的[问题][3],感觉没什么好的答案，没有深入去讲解的。不过还是贴一个：
> It doesn't matter what the reference is, it's the instantiated object's class that counts. The object that you're creating is of type A, therefore `this.getClass()` is always going to return A.

#### 小讨论
在OSC上的那个问题上跟人讨论一下。他说了
> 从AB强制类型转换为ABC。这是一个向上转型

我不赞同这种说法，因为实际上`this`指向就是一个`ABC`对象，不存在所谓的转型现象。也有代码可以证明这种行为:
```java
class ABC extends AB {
	@Override protected void test() { System.out.println("ABC"); }
	public ABC(){ }
	public static void main(String[] args) throws Exception{
		new ABC();
	}
}
class AB {
	protected void test() { System.out.println("AB"); }
	public AB() { this.test(); }
}
```
这里的输出会是`ABC`,即使不进行类型转换，this也还是会调用这个方法。

结论
----------------


题外话
----------------
总觉得我讨论问题的时候态度略不友善。

补充
------------
### 向上转型和向下转型
子类引用的对象转为父类，也就是向上。相反的，父类引用的对象转为子类就是向下。上个栗子
```java
Dog d = new Dog(); 
Animal animal1 = (Animal)d; //向上
Object o = new Dog(); // 向下
Dog d1 = (Dog)o;
```
向上转型通常都是成功的，但是向下转型却不一定。所以向下转的时候常用的操作就是用`instanceof`判断类型是否正确。  

### 关于`<init>`方法
从一篇不大靠谱的[文章][4]看到了`<init>`是如何生成的,它的初始化顺序本身就有问题。

[1]: http://www.oschina.net/question/941896_173532
[2]: http://www.cnblogs.com/lijunamneg/archive/2013/02/05/2893111.html
[3]: http://stackoverflow.com/questions/5155811/inheritance-and-the-this-keyword
[4]: http://www.javaworld.com/article/2076614/core-java/object-initialization-in-java.html
[5]: http://docs.oracle.com/javase/specs/jls/se7/html/jls-6.html#jls-6.6
[6]: http://docs.oracle.com/javase/specs/jls/se7/html/jls-8.html