title: GeeksQuiz的Java题目
date: 2014-09-23 23:04:57
tags: [GeeksQuiz, java, 问题]
---
刚好看到GeeksQuiz上有Java的题目，感觉会蛮好玩的，所以做了下。  
有些细节感觉自己真的没有注意到，稍微记录下。题目在这里：[GeeksQuiz][geekquiz]

<!--more-->

多态
-------------
> Java的`protected`标记的变量和方法是子类可见且**包可见**

一直以为`protected`跟`default`是不存在交集的，原来`default`只是`protected`的子集而已吗。
看下这个例子：
```java
class A {
    protected int x = 10, y = 20;
}
class B {
    public static void main(String args[]) {
        A a = new A();
        System.out.println(a.x + " " + a.y);
    }
}
```
输出正常且不会有任何错误。
>Java的**多态错误**会尽量在编译时提醒

虽然个人觉得是`Runtime Error`，但实际上是`Compile Error`。即使对于重载的方法，Java也会在编译时给予严格的类型检查。

```java
class Base {
    public void foo() { System.out.println("Base"); }
}
class Derived extends Base {
    private void foo() { System.out.println("Derived"); } 
}
public class Main {
    public static void main(String args[]) {
        Base b = new Derived();
        b.fun();
    }
} 
```

> Java不支持`super.super`,只能调用直接父类的方法。

不给例子了=。 =一般不会这么用。

数据类型
-----------
>Java的本地变量不会初始化

只有成员变量才会自动初始化。我跟数组搞错了= =
```java
class Main {   
   public static void main(String args[]) {      
         int t;      
         System.out.println(t); 
    }   
} // Complie Time Error
```

数组
------------------
> Java的数组不支持在声明时显式明确大小。

Java的数组声明时只能声明类型跟变量名，长度是在分配之后才决定的。一旦分配就会自动初始化。所以下面会直接报编译错误。如果加上`arr[] = new int[2]`就输出正常了。
```java
class Test {
   public static void main(String args[]) {
     int arr[2]; 
     System.out.println(arr[0]);
     System.out.println(arr[1]);
   }
}
```
final
--------------
> final永远只能被初始化一次。
 
注意一开始的自动初始化不算。
```java
class Main {
 public static void main(String args[]){
   final int i;
   i = 20;
   System.out.println(i);
 }
}
// 20
```

Exception
----------------
> 1. 能被抛出只能的是`Throwable`对象。
> 2. 被Catch的异常如果有多态，则子类必须在前

关于第二点，如果父类在前则会编译错误，因为子类永远不会执行。
```java
class Base extends Exception {}
class Derived extends Base  {}
public class Main {
  public static void main(String args[]) {
    try {
       throw new Derived();
    }
    catch(Base b) {}
    catch(Derived d) {}
  }
}
```


[geekquiz]: http://www.geeksforgeeks.org/java/