莫名其妙的问题
====
View的构造器
------
如果一个自定义的View没有重写CustomView(Context context, Attributes attrs)这个构造器就会出现这个异常：
```java
E/AndroidRuntime(20813): Caused by: java.lang.NoSuchMethodException: <init> [cl
ss android.content.Context, interface android.util.AttributeSet]
E/AndroidRuntime(20813):  at java.lang.Class.getConstructorOrMethod(Class.
java:460)
E/AndroidRuntime(20813):  at java.lang.Class.getConstructor(Class.java:431
) 
E/AndroidRuntime(20813):  at android.view.LayoutInflater.createView(Layout
Inflater.java:561)
```
想想，我应该是误解了重载的意思了。我总以为如果没有重写它也会调用父类的构造器，但父类构造器跟子类是完全不同的一回事啊。

ObjectAnimator
-----------
ObjectAnimator中需要更改的属性一定需要set跟get方法= =