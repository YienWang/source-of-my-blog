title: 【面试题】删除字符串
date: 2014-09-18 10:35:29
tags: [面试题,算法] 
---
> 删除字符串中的“b”和“ac”，需要满足如下的条件：
字符串只能遍历一次, 且不能够使用额外的空间
例如：
acbac ==> ""
aaac ==> aa
ababac ==> aa
bbbbd ==> d
进一步思考：如何处理aaccac，需要做哪些改变？

<!--more-->
一开始的想法非常简单粗暴。根据当前数组元素是什么来做分别处理：
1. 当前位置为a，设置一个bool值为true，代表接下来将根据下一位是否为 c 来做移位操作。
2. 当前位置为b，直接将b后面开始的所有元素都往前挪动一个元素，并将当前index减去2，长度减1。
3. 当前位置为c，如果之前设置的bool值为true，那么就同 b，将c后面的元素前移2位到ac，最后将index减去3，长度减2。
（index前移的位置要比移除的位置多1，是为了能够检查到aaccac这种删除后才出现ac的情况)

这样子其实算是遍历了多次了吧= =在最坏情况下会是 O(n2) 的算法复杂度，空间倒是满足了。
```c++
void filter(const char* arr) { 
    int length = strlen(arr) + 1;
	bool currentState = false;
	for(int i = 0; i < length; i++) {
		if (arr[i] == 'a') {
			currentState = true;	
		}
		if (arr[i] == 'c') {
			if (currentState) {
				for (int j = i - 1; j < length-1; j++) {
					arr[j] = arr[j+2];
				}
				i -= 3;
				length -= 2;
				currentState = false;
			}			
		}
		if (arr[i] == 'b') {
			for (int j = i; j < length - 1; j++) {
				arr[j] = arr[j+1];
			}
			i -= 2;
			length--;
		}
	}
}
```
那么遍历一次的情况要怎么做？  
上面的做法中，是遇到一个该删除的字符时就将后面的元素全部往前拷贝。实际上并不需要‘删除’，而可以试着‘忽略’中间不符合的元素。也就是用两个index来记录删除前后的字符串，其中一个index是遍历原本的字符串，而另外一个index（curIndex) 则是记录那些将被复制到新字符串的位置。  

**基本的思路如下：**
1. 如果上次字符的结尾为 a：
**1.** 如果下一字符不是c，那么可以将上次字符的a复制到curIndex，且curIndex自增   
**2.** 在1的条件下，如果下一字符不是a或者b，那么也将当前的index的元素复制到新字符串。
2. 如果上次字符的结尾不为 a: 如果当前元素不为a和b，那么可以将当前index元素复制到curIndex
3. 在curIndex大于1时，增加一检测：如果上一元素和上上元素组成了'ac',那就将curIndex自减去2.
4. 最后记得，我们在遇到当前字符为a的时候并未拷贝该元素。那么在退出循环的时候，最后元素为a，那么我们要将其加回来。
代码如下：
```c
void stringFilter(char* str) {
	bool isPreA = false;
	int curIndex = 0;
	for (int i = 0; str[i] != '\0'; i++) {
		if (!isPreA && str[i] != 'a' && str[i] != 'b') {
			str[curIndex] = str[i];
			curIndex++;
		}
		if (isPreA && str[i] != 'c') {
			str[curIndex] = 'a';
			curIndex++;
			// 如果当前字符不是a，且不是b，那么就直接拷贝当前元素
			// 不能是a的原因：我们只能在遇到a的下次循环才将它拷贝
			if (str[i] != 'a' && str['i' != 'b']) {
				str[curIndex] = str[i];
				curIndex++;
			}
		}
		// 这里的考虑一下：aaccac，在忽略了中间的一个ac之后，剩下一个ac，curIndex指向了c下个元素，那么只要上一
		if (curIndex > 1 && str[curIndex-1] == 'c' && str[curIndex-2] == 'a') {
			curIndex -= 2;
		}
		isPreA = (str[i] == 'a');
	}
	if (isPreA) {
		str[curIndex] = 'a';
		curIndex++;
	}
	str[curIndex] = '\0';
}
```
