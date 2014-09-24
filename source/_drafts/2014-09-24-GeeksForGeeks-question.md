title: GeeksForGeeks
date: 2014-09-24 09:39:38
tags: [GeeksForGeeks, 算法]
---
数组
-------------
> 求两个有序的数组的交集和并集

如果不考虑原来的数组存在重复元素，会是蛮简单的一道题目。只需要遍历一次两个数组就能够解决问题。但是如果存在重复元素，问题就会麻烦得多。(后面那种情况待考虑）
```c
void printIntersection(int arr1[], int N, int arr2[], int M) {
	int i = 0, j = 0;
	while (i < N && j < M) {
		if (arr1[i] < arr2[j]) i++;
		else if (arr1[i] > arr2[j]) j++;
		else {
			cout << arr1[i++] << " ";
			j++; }
		}
	}
}
void printUnion(int arr1[], int N, int arr2[], int M)  {
	int i = 0, j = 0;
	while (i < N && j < M) {
		if (arr1[i] > arr2[j]) 		cout << arr2[j++] << " ";
		else if (arr1[i] < arr2[j]) cout << arr1[i++] << " ";
		else {
			cout << arr1[i++] << " ";
			j++;
		}
	}
	while (i < N) cout << arr1[i++] << " ";
	while (j < M) cout << arr2[j++] << " ";
	cout << endl;
}
```

> 求三个有序数组的共同元素（包括重复）

跟求两个有序数组并集类似的思路，遍历三个数组，如果出现了 `ar1[i] < ar2[j]`，自增i，因为这个时候的`ar1[i]`不可能是共同元素。同理的，如果 `ar2[j] < ar3[k]`，那么`ar2[j]`也不能是共同元素。如果均不符合，那么就有 `ar1[i] > ar2[j]， ar2[j] > ar3[k]`,`ar3[k]`不会是共同元素。
```c
void printCommon(int A[], int B[], int C[], int X, int Y, int Z) {
	int i = 0, j = 0, k = 0;
	while (i < X && j < Y && k < Z) {
		if (A[i] == B[j] && B[j] == C[k]) {
			cout << A[i] << " ";
			i++; j++; k++;
		} 
		else if (A[i] < B[j]) i++;
		else if (B[j] < C[k]) j++;
		else k++;
	}
}
```

字符串
------------------------
> [寻找两个字符串的最长子串][longest_substring]

跟下面的模式匹配问题有关，或者使用动态规划。

> 字符串模式匹配

几种经典的方式：KMP，Robin-kraft，boyer-morer，还有[suffix tree][suffix_tree]

[longest_substring]: http://www.geeksforgeeks.org/longest-common-substring/
[suffix_tree]: http://www.geeksforgeeks.org/pattern-searching-using-trie-suffixes/
[suffix_tree_intro]: http://www.geeksforgeeks.org/pattern-searching-set-8-suffix-tree-introduction/


