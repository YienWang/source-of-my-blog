title: Python的快速入门-《A Byte of Python》
date: 2014-10-14 14:57:46
category: 读书
tags: [python,笔记,入门,我的思考,问题]
---

正文
-----
### 内容简介
虽然这本书说适合没有太多编程基础的人，但我这篇总结是为具有一定编程基础的人所准备的- -
书中一开始先简单介绍了python的一些基本语法规则，标识符，接着按章节以‘**操作符与表达式**’，‘**控制流**’，‘**函数**’，‘**模块**’，‘**数据结构**’，‘**OOP**’，‘**输入和输出**’，‘**异常**’及‘**其他拓展**’的顺序较为完整地介绍了python的主要内容。
<!--more-->
### 基础
- python只有int，没有long。int但可以表达任意的大小
- 单引号效果跟双引号类似。但两组三引号之间可以有任意行的内容。(常用于模块或类说明内容）
- `format()`是个好方法。
- python的缩进非常非常重要，一旦搞错就有可能报错。

```python
# encoding=utf-8 
print ("{0:*^9}".format('hello'))     # 将format的内容用^前的符号填充到指定长度
print ("{0:.3f}".format(1/3.0))       # 取小数点后三位输出
print ("value %r %d" % (10, 10))      # r for everthing
print ("value {} {}".format(10, 10))
print ("value", "{}".format(10))      # ','相当于空一格
print (r"raw_string\n")               # 加上r表示不发生任何转义,适用于正则
"""
**hello**
0.333
...
raw_string\n
"""
```
### 操作符与表达式
- 除了其他语言常见的操作符及用法，python还有自己独特的地方
- `*`可用于一个数字与一个字符串，用于复制相同的字符串n遍
- `**`用于计算幂
- `not`用于取反  【SELF】：注意bool是int的子类= =所以0是False
- `and`用于与操作，支持短路求值
- `or`用于或操作，支持短路求值
- 赋值具有右结合性：`a=b=c`即`a=(b=c)`
- 不支持自增自减符号。`--a`只代表取反两次，`a--`直接报错

### 控制流
判断：
```python
# encoding=utf-8
isRight = False
while not isRight:
    value = int(raw_input("input value:"))
    if value == 3 : isRight = True
    elif value < 3: print "larger"
    else: print "smaller"           # else跟冒号间不能加任何内容

print ("Guess Right")
```
循环：
```python
# while的例子上方有
for i in range(1, 5): # 1 2 3 4
    print i,
for i in range(1, 5, 2): # 1 3 第三个参数控制步长
    print i,
for i in [1,2,3,4,5]: # 1 2 3 4 5
    print i,
for i in list(range(1, 5)):
    print i,
# 其他break和continue的控制流语句类似
```
### 函数
局部和全局变量
```python
# encoding=utf-8
x = 50
# 没有引入全局变量x，引用的是局部变量
def func_local(x, value):
    print x # 50
    x = value
# 全局变量需要先声明再使用，否则有warning: name 'x' is used prior to global declaration
# 无法定义 global x = 2
def func_global(value):
    global x
    print x # 50
    x = value
func_local(x, 2) # 50
print x 	  # 50
func_global(2) # 50
print x # 2   # 2
```
默认参数值
```python
# 默认参数只能是参数列表的最后一个或最后几个参数
def callHello(content, times = 1):
    print (times*content)
callHello("hello")  # hello 
```
关键字参数(Keyword Arguments)
```python
# 仍然是默认参数值，只是调用函数时的一种技巧
def callHelloTo(content, someone="you", user = "I"):
    print ("{} say {} to {}".format(user, content, someone))
callHelloTo(someone="Brain", content="Nice to Meet you")
# I say Nice to Meet you to Brain
```
变长参数
```python
# numbers是list，而kcount是dictionary
def calsum(init = 0, *numbers, **kcount):
   '''cal input value sum'''
	result = init
	for value in numbers:
		result += value
	for key in kcount:
		result += kcount[key]
	print ("result is {}".format(result))
calsum(1,2,3,4,5,d=-15) # result is 0
```
return语句
- 如果没有写return语句，默认是返回`None`

DocStrings
- python的注释文档，即在函数体/模块/类的首行三引号间的内容。
- 可以通过`print xx.__doc__`或`help(xx)`输出

### 模块
- 模块即可复用的部分。 
- 最简单的是直接写一个py文件，直接`import`导入
- 也可使用编译后的pyc文件（字节码形式，而非机器码，因而是平台无关）
- 可通过`__name__`获取模块名称。常用`__name__=='__main__`判断是否主模块。
- 导入时也可使用`from ... import ...`,只是不建议，可能命名冲突。
- 也可使用`from ... import *`的形式导入所有公有域和公有方法 
- 定义一个通用的module通常需要遵守一些公有约定，比如私有域中包含版本信息，作者信息等。
- 可以通过`dir()`方法获取模块所有域的信息。括号中不包含模块即代表查询本模块。
- package是module的集合。在每个模块上都有一个\_\_init\_\_.py文件说明模块信息

### 数据结构
Python内置四种数据结构，list，tuples，dictionary，set。  
- list是可变的，即列表可增删，排序等。
- tuple是不可变的，在初始化之后即不可再改动
- dictionary是key-value，添加时不能保证有序。key必须为不可变对象
- set是简单对象的无序序列，可用于集合操作（set本身就支持集合操作）

关于sequence
- list，tuple，string都是sequence，都具有集合操作（比如`in`和`not in`等），另外也支持下标取值。下标可以为负值，代表逆方向取值。
- sequence也支持slicing操作，即`[:]`和`[::]`。通过该操作返回的是原sequence的一个拷贝。可以通过`[::-1]`轻易获得一个逆序。

关于引用
- 同样存在别名现象，所以如果需要复制数组，可以使用slicing（即`[:]`)

### OOP
- method和field是class的attribute，两者均有实例和类两种范围。
- 类的每个实例方法的第一个参数都是self，即自身。可以理解成是Java和C++中的this。【作者给出的是这种说法真有趣，事实是否真的如此？:`myobj.method()=>MyObj.method(myobj)`】
- 构造器：`__init__(self,...)`。如果存在基类，则先调用基类的初始化函数
- 类方法和静态方法看做是Python decorator的使用
- Python中类所有的method和field都是virtual的
- 继承，所有基类的构造函数都需要自己调用

一个完整的例子：
```python
# encoding=utf-8
class Person:
    '''A person is the base of everyone'''
	count = 0                 # 类变量在外部声明
	def __init__(self, name):
		self.name = name
		Person.count += 1     # 使用是使用类引用变量
	@classmethod              # 也可以用staticmethod，但调用时再传入类名。
	def getCount(cls):
		print ("Total: {}".format(cls.count))
	def call(self):
	   '''Show person info'''
		print ("Name is {}".format(self.name))
# Python也支持多重继承，只需要在声明时增加类即可
class Student(Person):
	def __init__(self, name, school):
		Person.__init__(self, name) # 先调用基类初始化。没有super
		self.school = school
	def call(self):
		Person.call(self)           # 调用基类方法时也需要传入self
		print("{}'s shool is {}".format(self.name, self.school))
```
### 输入和输出
- `raw_input()`用于获取用户输入
- `open("file", [options])`操作用于读写文件。file的`read()`,`readline()`等操作可以用于读行的内容。
- `Pickle`这个module可将python的对象保存到文件中。只需要使用`pickle.dump(data, file)`和`pickle.load(file)`操作~
- 对于unicode字符，需要使用`encoding="utf-8"`

### 异常
- 使用`try...except...`捕获异常
- 使用`try...finally...`保证即使发生异常也能够好好收拾残局-->更加推荐的方式是`with...as...`。

```python
class ShortLengthException(Exception):
    def __init__(self, length, content):
        Exception.__init__(self)
        self.content = content
        self.length = length

    value = raw_input("Input-->")
try:
    if len(value) < 3:
        raise ShortLengthException(len(value), value)
except EOFError:
    print ("Not an EOF!")
except ShortLengthException as ex:
    print ("exception: {} 's length {} is to short"\
	   .format(ex.content, ex.length))
else:
    print ("nice length")
```
### 更多
- **Passing tuples around**:可将tuples整个作为返回值：
```python
def get_error_detail():
    return (2, "error")
errnum, errstr = get_error_detail()
# 最快的交换方式-Why can: 后面的参数先被创建为一个tuple再进行赋值
a, b = b, a
```
- ** Special method**：除了`__init__()`这个方法外，还有其他类似的内置函数可用于增强功能。如实现`__getitem__()`即可实现任意对象下标取值。其他还有`__str__()`，`__len__()`等等...
- **Single Statement Blocks**：`if True: print ("Nice")`
- **Lambda Forms**：（讲得不大清楚= =)
- **List Comprehension**：
```python
c = list(range(1, 100))
result = [i for i in c if i % 2==0 or i % 9 == 0]
print result # 100以内所有能被2或9整除的数
```
- **Tubles and dictionaries as arguments**：前面已使用
- **Assert statement**：并不少见
- **Decorators**：仍然解释不清楚

### 我的问题
- lambda表达式是怎么回事= =
- Python的decorators到底是什么

正文无关：
------------
### 为什么选这本书
前几天一直在网上看*《Learn Python In A Hard Way》*的在线教程，效率实在是低---这书讲得真的是太慢了，每次都只教一点点零碎的这种学法不适合我。  
我想要看的是那种能够快速了解语法，能够快速上手的书。而《A Byte of Python》非常地和我胃口，事实证明也是如此~  
看完这本书之后，下一步的阅读计划就是*《Python Cookbook》*。这本Cookbook主要是一些代码片段，当然通常是很赞的技巧~看起来非常有趣，现在翻阅起来基本无障碍>_<~

### 如何学习一门新语言
在我的书签中翻到这样的一篇博客["如何学习一门新的编程语言？——在学习区刻意练习"][1]。上面推荐了*《Learn XXX In Hard Way》*系列，作者认为带有大量习题的教程更加适合编程语言的入门。  
而经过这次Python的学习，我个人并不赞同这种说法。作为一个有不少编程基础的非新人，接触一门新语言还需要从Rookie级别的书看起其实是费力不讨好的。 
看了*《Learn Python In Hard Way》*的十几个Exercise，我还是停留在几个print语句上，累感不爱。而*《A Byte Of Python》*用了130页左右介绍了Python中的大部分内容，书的结构简单明了。知道这些90%的内容之后就可以干活了。实际使用的机会越多，越熟练，而不是做重复做那些习题。

[1]: http://www.yangzhiping.com/tech/learn-program-psychology.html



