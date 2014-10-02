title: 技术书单
date: 2014-09-21 01:23:49
tags: [书单,读书]
---
最近从电脑里面翻到了本电子书，看起来特赞。现在想想，我是不是忽略了很多好书，所以就来写个清单文件~有个start/end,end之后还要有一篇blog。
<!--more-->
Android
-----------------
### ***《Android Programming Pushing Limit》*** 
**start：**14-09-20
> 粗略地翻了一下目录。应该说这本书讲得特别地全，但不乏深度。前面是些基础，但也不会基础到教你一个控件怎么用。后面开始会介绍一些应用开发过程中会遇到的场景，教你怎么使用hide的API。最让人惊讶的是这本书里面竟然有提到怎么使用`Volley`和`OkHttp`，虽然点到即止，但是简洁明了。这本书真心值得细看。

### ***《Efficient Android Threading》*** ### 
**start：**14-07-16
> 一本将Android的多线程机制讲得非常透彻的书。应该覆盖了Android里面所有的异步以及多线程框架。  
> 读这本书的过程中，它让我知道了一个很重要的“常识”。**使用AsyncTask是非常具有风险的**（有一种说法叫做 *AsyncTask is evil* )。基本上那些讲到`AsyncTask`却没提说它很可能导致Activity/Fragment泄露的，那本书很大可能是烂书（国产书籍可作证）。

### ***《Android开发精要》*** ### 
**start：**14-04-14
> 应该是国内讲Android应用方面算非常好的一本书（虽然作者对Android那些专用名词的翻译真心奇葩= =)。有些内容会偏向于框架层，但基本还是集中在应用层开发方面。  
> 这是第一本我觉得写的很赞的国人的书。（第二本是老罗的源码分析--只是那本书偏向于底层太多，而且都是跳来跳去地读代码，读起来甚是乏味，能力不够，待打怪升级后再次挑战）
> 前阵子是翻完了这本书，只是有部分内容需要回味一下。届时再做总结。

Java
-------------------
### ***《Java编程思想》*** ### 
**start：**14-05
>可以算是Java宝典？个人觉得不是新手向的，虽然书的封面有读者的评论说“通俗易懂”。
>关于泛型和协变类型仍然有点晕。或者我连数组也没搞懂？

### ***《Effective Java》*** ### 
**start：**14-06 **end：**~~14-08~~
> 关于有效提升Java开发效率的点。主要在讲如何使用设计模式，如何设计接口方面（现在再翻一遍应该更有体会）
> 09-22:我很怀疑我之前到底是怎么读这本书的。重新再翻，感觉完全不一样啊。我有种在看一本从未看过的书的赶脚！

Blog: [Ch2][effect_java_ch2]
Programming
-------------------
### ***《七周七语言》*** ### 
**start：**unknow
> 以前曾经翻过这本书，当时看起来真心乏味。那个时候我还无法理解FP为什么会存在。  
> 在现在大致了解了Scala，以及各种框架之后，重新翻这本书很是有趣。只有一周基本做不了什么，但是它能让学到一门语言最经典的地方。Scala是面向对象与FP的有机融合，Clojure是类Lisp的FP。Haskell是一门我现在仍理解不了的纯FP语言。但是会懂的，虽然不是现在。  
> 打怪练级，然后看看这本书会有很多收获~

Computer Science
------------
### **《深入理解计算机系统》** ### 
**start：**09-25
> 好书不解释~s

Other
-----------------
### ***《RFC2616 -HTTP1.1》 *** ### 
**start：**14-09-22 
>要深入了解一个协议，最好的方式就是去看RFC~突然在pad上找到了之前从网上下载的一份翻译版的HTTP1.1说明【偶尔还是想看看中文的-__-】  
>有些内容笔试的时候还问到过，比如哪些方法是安全的。


[effect_java_ch2]: {{root_url}}/2014/09/24/blog-for-effiective-Java-ch2/