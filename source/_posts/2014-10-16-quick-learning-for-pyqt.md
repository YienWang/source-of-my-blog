title: PyQt的入门之旅
date: 2014-10-16 14:36:42
category: 自学
tags: [python, Qt, 入门]
---
前言
---------
PyQt（Pai Qt）是使用Python写的Qt框架，基本是使用Python对Qt的核心模块进行封装，再使用SIP调用这些C++模块。  
对于Qt，我之前是非常不喜欢的。QtC++的IDE真心难用，这让我对这个优秀的跨平台框架一直不怎么感冒。而PyQt似乎只用到了IDE中的QtDesigner部分而已，这样的话就有兴趣学一下了。

<!--more-->
正体
--------
### 安装
安装起来算是有点简单，如果你没搞错系统的位数和对应的安装包的话。Windows平台并不需要编译，有现成的binary包可以使用。     
先去下载一个Python：如果想用Qt5.0的话就必须使用python3.4（好累，真不想装多版本）。  
装完python之后再装pyqt。完成之后，先尝试一下命令：
```
>>> import PyQt5
>>> help(PyQt5)
```
如果没报错的话那就是安装成功了。接着打开PyQt5文件夹中qtcreator，随便创建一个界面文件，保存。在命令行中执行下面转换的指令：
```cmd
pyuic5 -x -o xx.py xx.ui
```
然后我就出现这样子的错误T^T:
```
ImportError: DLL load failed: %1 is not a valid Win32 application
```
最后发现是机器位数不对应。机智如我，以为是官方的64位不正常，跑到了非官方网站下载64位的PyQT4。（好累，刚刚才发现是我下错包了T^T)重新下一个64位的包，删掉PyQt4，再执行上述命令的时候就正常了。

### HelloWorld
上面提到了在命令行中敲下面几句能够将对应的ui文件转换为py文件：
```cmd
pyuic5 -x -o file_output.py file_input.ui # -o 代表需要输出文件
```
这里加上`-x`指令可以在生成的py文件中添加main入口，但**非常不建议**这么做。  
每次对于Ui文件的修改都需要重新生成py文件，而这将会覆盖原本的代码。很明显的，Ui的文件是会经常变动的，所以不能在这里写逻辑代码。  
通用的做法是另外创建一个文件将界面import进来再做处理：  
```python
from PyQt5.QtWidgets import QApplication, QMainWindow
# module名也就是文件名
from Hello import Ui_MainWindow

class HelloworldBase(QMainWindow, Ui_MainWindow):
    def __init__(self, parent=None):
        super(HelloworldBase, self).__init__(parent)
        self.setupUi(self) # 设置ui界面，调用的是布局文件中的相关逻辑

class HelloWorld(HelloworldBase):
    def __init__(self, parent=None):
        super(HelloWorld, self).__init__(parent)
        # 通常信号槽的连接操作比较多，一般这里会被重构为函数
        self.btn1.clicked.connect(self.callHello) 
	
    def callHello(self):
        print ("HelloWorld")
        self.pushButton_2.setText("Hello World")
		
if __name__ == "__main__":
    import sys
    app = QApplication(sys.argv)
    app.setApplicationName("hello")
    world = HelloWorld()
    world.show()
    sys.exit(app.exec_())
```

待补充
-------------
关于PyQt信号槽的操作会再细说，现在先跑跑HelloWorld吧~

补充
-----------
### Python 3.x
- Python 3.x 开始对语法检测严格了很多，空格用于缩进，如果使用tab之类的就会报错误。因此需要在NotePad++中设置将tab等效为空格。错误详情：`TabError: inconsistent use of tabs and spaces in indentation`
- 对于输出，也无法直接使用双引号或单引号，必须使用空格，否则会报错：`SyntaxError: invalid syntax`

### PyQt5 不再支持Phonon
- 从PyQt5.x开始，多媒体都是使用`QtMultimedia`这个类来做了，具体可以看官方的demo（默认会是在`\PyQt5\examples\multimedia\audiodevices`）


### QtDesigner使用小技巧
- Layout填充整个窗口：拖进去一个Layout，取消焦点，再在Window中右键，选择布局--垂直或者水平均可以。
- Ctrl+R 窗口预览

参考资料
----------
[官方提供的PyQT5下载][pyqt5]
[64位的PyQT4非官方下载][2]
[Youtobe上PyQt的简单教程][1]
[Python 3.x中关于tab和spaces的说明][tabs]
[StackOverflow上关于ImportError错误的答案][3]

[1]: www.youtube.com/watch?v=GLqrzLIIW2E&list=UUPme28sMOcWS50CgtTWUZIw
[2]: http://www.lfd.uci.edu/~gohlke/pythonlibs/#pyqt
[3]: http://stackoverflow.com/questions/4381936/pyqt4-and-64-bit-python
[pyqt5]: http://www.riverbankcomputing.com/software/pyqt/download5
[tabs]: http://legacy.python.org/dev/peps/pep-0008/#tabs-or-spaces
