title: 读《NodeJS开发指南》
date: 2014-10-06 08:44:45
tags: [nodejs, 读书, 笔记]
---

章4
---------
- JavaScript的面向对象特性是基于原型的，而不是基于类。即通过原型复制来实现对象继承的特性。

问题
------------
- `process.nextTick(callback)`为什么会比`setTimeout(callback,0)`更有效率？