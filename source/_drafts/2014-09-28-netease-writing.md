网易笔试题
------
### 音频字节计算
> 使用44.1kHz采样率，立体声，16比特模式采样的1分钟音乐占多少空间

计算`44100 * 2 * 16 / 8 * 60 = 10584000byte = 10.09M`

Android支持的音频格式：

### Linux重定向
> ./a.out > outfile 2>&1 的含义

执行`a.out`并将标准的输出和标准的错误内容重定向到outfile中（即写入）。
这里`>`代表重定向，`&`代表等同于，数字0,1,2分别代表标准输出，标准输入，标准错误。  
`2>&1`的意思是将标准错误也输出到标准输出当中。

### 矩阵运算
> 写出3*3的绕y轴旋转的矩阵。

先从2*2开始
```
(Rcos(a+b), Rsin(a+b)) 
= (Rcos(a)cos(b)-Rsin(a)sin(b), Rcos(a)sin(b)+Rsin(a)cos(b)
because: (x,y)=(Rcos(a),Rsin(a))
= (xcos(b) - ysin(b), xsin(b) + ycos(b))
```
由此可得，对2*2方程，旋转矩阵为`{{ cos(b), -sin(b)}, { sin(b), cos(b)}} `
对于3*3绕y轴的矩阵，则有：`{{cos(b), 0, -sin(b)},{0, 1, 0},{sin(b), 0, cos(b)}}`
### 数据库查询语句
> 从Student表中，按照age的所属人数按照递减的顺序获取数据

`COUNT(*)`用于获取每个分组所拥有的人数。如果除了`COUNT(*)`还有其他要获取的属性，则`GROUP　BY`一定要加上，否则会报错。
```sql
SELECT age, COUNT(*) AS numbers
FROM  `student` 
GROUP BY age
ORDER BY numbers DESC 
LIMIT 0 , 30
```

### 数据库查询引擎
> 写出两种常见的数据库查询引擎

纯常识性的内容，只是很少接触MYSQL这块的内容。搜了一下，有`MyISAM`和`InnoDB`  
似乎常用的是后面那个
> InnoDB给MySQL提供了具有提交、回滚和崩溃恢复能力的事务安全（ACID兼容）存储引擎

一个简单的比较：
MyISAM适合：(1)做很多count 的计算；(2)插入不频繁，查询非常频繁；(3)没有事务。InnoDB适合：(1)可靠性要求比较高，或者要求事务；(2)表更新和查询都相当的频繁，并且表锁定的机会比较大的情况。[参考资料][mysql_engine]

### 虚函数
简而言之，C++的虚函数是通过虚函数表来实现。对于有虚函数的类，其实例和子类实例维护一个虚函数表。如果子类覆盖了父类的虚函数，则该位置的函数指针就会指向子类的函数实现地址。[参考资料][virtual_function]

[mysql_engine]: http://www.cnblogs.com/sopc-mc/archive/2011/11/01/2232212.html
[virtual_function]: http://blog.csdn.net/haoel/article/details/1948051