title: strange input output in C
date: 2014-09-23 22:46:36
tags: 
---
**Q:**考虑输出
```c
// Assume base address of "GeeksQuiz" to be 1000
#include <stdio.h>
// Assume base address of "GeeksQuiz" to be 1000
int main()
{
   printf(5 + "GeeksQuiz");
   return 0;
}
```
**A: **输出是 `Quiz`。竟然忘记了后面那个是字面常量，它代表是常量池中的地址。所以上面的操作相当于将数组的初始位置右移五个位置，再打印字符串。

**Q:**考虑输出
```c
#include <stdio.h>
int main() 
{ 
  printf(" \"GEEKS %% FOR %% GEEKS\""); 
  getchar(); 
  return 0; 
}
```
**A: **输出是 `"GEEKS % FOR % GEEKS"`。`%`一般情况下是用来输出的，如果使用`%%`才可以输出这个符号。

