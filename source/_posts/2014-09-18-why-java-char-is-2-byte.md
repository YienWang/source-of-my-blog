title: "为什么Java的char是两个字节"
date: 2014-09-18 23:00:46
tags: [java,编码,问题]
---
之前就有一个疑问:
>为什么我在String里面存了中文，长度仍然是中文的个数？

<!--more-->

这个问题看起来很蠢吧，但是仔细想想会发现会很奇怪。单个字母会跟单个汉字等长度吗？考虑一下拼音的情况，五笔也是，每个字都由多个字母组成的。  
后来了解到了Unicode--字符的统一标准。查了下Java就是用UTF-16来表示字符的。汉字跟英文等长度了。  
一开始的问题算是解决了。但是又有一个问题，一个char是2字节，也就是16位。但是2的16次方等于65536。
>...最新的GB 18030-2005收录70244字（其中包括大量的东亚文字）[WIKI][wiki_hanzi]

先不说包含其他国家的字符，按照上面的字数统计，16位连汉字都无法完整表达吧。不过实际上很多汉字是比较少用到，那么如果用16位来表示最基本的字符集如何？这也就是Unicode多语言文字平面（Multilingual Plane）的存在的理由了。
BMP(Basic Multilingual Plane)是最基本的多语言平面，字符值区间为 `U+0000-U+FFFF`。除了BMP之外，其他还有SMP（多文种补充平面）具体见[字符集][wiki_unicode_mp]。每个平面都代表了一定的字符集，每个平面都有65536个 **Code Point(代码点）**

那么16位所能代表也就是刚好是这个BMP所能表达的字符数了。但官方的说明是：
> Java支持unicode 4.0 标准，该标准包含从 U+0000-U+FFFF 的基本多语言平面和 U+10000-U+10FFFF 的扩展平面的文字

好了，问题又来了。**为什么Java能够使用16位表示增强字符集？**
先说结论：**如果是增加字符集中的字符，Java会用两个char来表示**,也就是用32位来表示了。为什么会这么做，可以参考文章后面的参考资料。  
表示看完之后，才发现原来当初会这么做也是考虑了很多的啊~比如说一开始有个方案是创建一种新类型char32，不过考虑到向前兼容就不采用了。  
增强字符集的字符也可以用一个`int`来表示，这个倒是才发现，比如有这种用法：
```java
String s = new String(Character.toChars(codePoint));
```
使用后面那个方法无论CodePoint是不是增强字符集中的元素，都会返回一个char数组。
看下这个例子：
```java
String s ="𠀕";
char[] c = s.toCharArray();
byte[] b = s.getBytes();

System.out.println(c.length);
System.out.println(s.length());
System.out.println(b.length);
```
输出是 `2 2 4`   
这里我使用的中文字符是增强字符集中的 `U+20015`，这里的char数组长度就变成了2。 字符串的长度其实就是String中char数组的长度，因而前面两个输出都是2，utf-8的输出为4 byte，这点就不解释了~（自己对应一下范围就知道了）
### **如此看来还真不能想当然地认为中文的长度就是数组的长度**。我一开始的那个问题似乎前提就错了=。 =

补充
============
Code Point Or Encoding？
---------------
Code Point跟Encoding是不同的概念。Code Point是在计算机内的表示，而Encoding则是呈现的时候的编码。两者关注的点不一样。Code Point更多是在计算机表示上的统一，使用同一长度易于计算位置。而Encoding则更多追求空间上的效率。比如UTF-8的规则，如果是拉丁字母系，单个字符只需要一个字节就能够表示，而UTF8最长也就到6字节。如果用UFT-16，很多拉丁字符会产生多个无用的0（8个0)。（个人拙见）。
看下这个问题：
```java
String s ="中";
String sa = "A";
char[] c = s.toCharArray();
byte[] g = s.getBytes("GBK");
byte[] b = s.getBytes();
byte[] sab = sa.getBytes();
System.out.println(c.length);
System.out.println(g.length);
System.out.println(b.length);
System.out.println(sab.length);
```
在我的电脑上（默认UTF-8），上面的输出结果是 `1 2 3 1`  
意思也就是 中 在String里面作为1个char存在，使用gbk表示是2byte，使用uft-8表示是3byte，而字母 A 用utf-8表示1byte。


UTF-8 Or UTF-16？
---------------------
上面提到这两个编码规则的时候其实已经说到了两个编码规则适用的场所。选择什么字符集要看情况，现在网络上大部分是用UFT-8来作为基本的编码格式了~UFT-16更多是在编程语言中用来表示字符，比如PHP6似乎也要用UTF-16来编码了[look_this][php_utf16]
这里翻译一下stackoverflow上关于这两种编码规则的对比：
> UTF-8
> 1. 优点：  
> **1** 基本的字符(ASCII)可以原封不动（因为在UTF8中它们的表示还是不变的）。提供了较好的向前兼容；  
> **2** 不会存在null byte（全0）。这是由UTF-8特殊的编码规则决定的。每个byte的开头起码会有一个10。
> 2. 缺点：
> 一些常用字符可能有不同的长度。这导致在取址及在计算字符串长度的时候会很麻烦。
> UTF-16:
> 1. 优点：常用的文字能够用2字节来表示（也就是BMP）。也就是说UTF-16能够用来作为固定长度的编码，因而提高取址的速度。
> 2. 缺点：对应的ASCII码在UTF-16会有null byte.这在空间上会是一个损失。
> --[stackoverflow][difference_between_utf_8_and_16]


参考资料：
[java char support][java_char_support] : Java官方关于字符集实现的介绍。

[difference_between_utf_8_and_16]: http://stackoverflow.com/questions/4655250/difference-between-utf-8-and-utf-16
[php_utf16]: http://schlueters.de/blog/archives/128-Future-of-PHP-6.html
[wiki_hanzi]: http://zh.wikipedia.org/wiki/%E6%B1%89%E5%AD%97
[wiki_unicode_mp]: http://zh.wikipedia.org/wiki/Unicode%E5%AD%97%E7%AC%A6%E5%B9%B3%E9%9D%A2%E6%98%A0%E5%B0%84
[java_char_support]: http://www.oracle.com/technetwork/articles/javase/supplementary-142654.html