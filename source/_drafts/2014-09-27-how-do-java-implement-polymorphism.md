title: Java的多态是如何实现的
date: 2014-09-27 08:05:18
tags: [面向对象, java]
前言
-----------------
关于Java的多态，似乎没有多少人去研究过（不像C++，虚指针似乎成为面试的标配）。那么Java的多态到底是怎么实现的。下面代码又是在编译的时候发生了什么变化？
```java
public class Derived extends Base {
  @Override String toString() { return "Derived"; }
  public static void main(String[] args) {
    Base b = new Derived();
	System.out.println(b.toString());	
  }	
}
```
<!--more-->
正文
----------
### Java的方法调用方式
Java的方法调用方式有两种，动态方法调用和静态方法调用。前者是调用需要有方法调用所作用的对象，而后者是指对于类的静态方法的调用。  
类调用（Invokestatic）是编译时就被决定的，而实例调用（Invokevirtual）则是在调用时才确定具体的调用方法。  

### 方法表和方法调用
方法表是Java实现动态调用的主要方式（其他还有什:reflect&invokemethod）。针对于每个类，方法表被存储在方法区，包含类型所定义的所有方法和指向这些方法代码的指针。  
以一开始提到的代码为例。在调用`b.toString()`方法时，首先先算出`toString()`方法在b的方法表中的偏移量，然后根据实例方法调用的参数`this`得到具体的对象，从而得到该对象所指向的方法表，再根据刚才计算得到的偏移量调用对应的方法即可。

### 接口调用


Java的多态实现原理跟C++基本类似，但从底层上来说，Java会涉及到JVM的部分字节码指令，因而很少人会深入去研究。

静态方法和非静态方法调用的时候，实际上也就是两条Java字节码`Invokevirtual`和`Invokestatic`的区别。  
先说前一条指令所做的事。

[java_polo]: http://content.hccfl.edu/pollock/Java/Polymorphism.htm
[deep-in-jvm]: http://www.artima.com/insidejvm/ed2/jvmP.html