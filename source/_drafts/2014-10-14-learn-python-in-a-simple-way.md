title: Python的简单入门
date: 2014-10-14 14:57:46
tags: [python,笔记,入门]
---
前言
---------
前几天一直在网上看《Learn Python In A Hard Way》的在线教程，效率实在是低，感觉自己不适合那种完全新手向的书。这书讲得真的是太慢了，一点一点零碎的语法不适合我这种人。  
我想要看的是那种能够快速了解语法，能够快速上手的书。就目前看来，《Byte Of Python》非常地和我胃口。

正文
---------------
### 输入输出
```python
value = int(raw_input("Input value:"))

print "value %r %d" % (value, value) # r for everthing
print "value {} {}".format(value, value)
print "value", "{}".format(value) # ','相当于空一格
print r"raw_string\n" # out: raw_string\n 不会发生任何转义,适用于正则
```

### 控制流
判断：
```python
value = 3
if value == 3:
	print "nice"
elif value < 3:
	print "larger"
else value > 3;
	print "smaller"
```
循环：
```python
running = True
while running:
	running = False
	print "hello"
# 1 2 3 4 5
for i in range(1, 5):
	print i,
# 1 3
for i in range(1, 5, 2): # 第三个参数控制步长
	print i,
# 1 2 3 4 5
for i in [1,2,3,4,5]:
	print i,

# 其他break和continue的控制流语句类似
```

### 函数
局部和全局变量
```python
// Warning: Python的编码默认不是utf-8，所以如果存在中文注释那么就需要设置编码集
x = 50
# 因为不是全局的所以需要传入参数
def func_local(x):
	print x # 50
	x = 2

# 无法定义 global x = 2
# 定义global变量需要在程序的开始，否则有warning
# SyntaxWarning: name 'x' is used prior to global declaration
def func_global():
	global x
	print x # 50
	x = 2

func_local(x) # 50
print x 	  # 50
func_global() # 50
print x # 2   # 2
```
函数支持默认参数【可选参数】
变长参数
`*name`自动转换为tuble，`**name`转换为key-value。

