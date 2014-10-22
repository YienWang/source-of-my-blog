title: 《Effiective Java》 ：枚举和注解
date: 2014-09-24 20:43:48
tags: [Java,读书,笔记,问题]
---
第六章 枚举和注解
----------


### 关于Enums
兼听则明。Java的`Enums`也是有开销的。参考资料上有关于这点的讨论。
看完这些讨论，我觉得我快疯掉了= =在froyo之前（2.3），davlik对`enum`支持不是太好，会导致该类占用大量空间。但在后来改进的版本中， 对于`enum`的处理得到了很好的提升。可是在Android的doc中仍然不是太建议使用，如何权衡。

还有个问题未解决，枚举真的会在多进程中创建多个吗。枚举不是只会创建一次的吗？


参考资料
------------------
[G+上关于Enum的讨论][enums_in_g_plus]
[Android官方说明的内存优化][android_memory]
[Android性能优化提示][android_performance]
[StackOverFlow上关于Android上Enum的问题][stack_enums]

[stack_enums]: http://stackoverflow.com/questions/5143256/why-was-avoid-enums-where-you-only-need-ints-removed-from-androids-performanc/ 
[android_performance]: http://developer.android.com/training/articles/perf-tips.html
[android_memory]: http://developer.android.com/training/articles/memory.html
[enums_in_g_plus]: https://plus.google.com/+AndroidDevelopers/posts/eKCexh3Sw1P
