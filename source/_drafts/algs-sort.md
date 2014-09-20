title: 常用算法
tags: [算法,c]
---
在网上看到一堆资料，但是觉得各种不直观。还是自己写一个好了。  
堆排序
---------------
```c
// 使堆有序的算法。注意这里是从0开始的索引。
void heapify(int arr[], int length, int pos) {
	// 2 * pos + 1 是左节点的位置 [0,1,2]<---不直观的话自己对应
	while ((2 * pos + 1) < length) {
		int child = 2 * pos + 1;
		// 如果当前节点非最后的节点（即存在右节点），则比较左右两节点
		if (child < length - 1 && (arr[child+1] > arr[child])) child++;
		// 比较左右字节点中最大的节点与当前节点，如果小于则退出循环
		if (arr[child] < arr[pos]) break;
		// 不满足堆有序定义，则交换两个节点
		swap(arr[pos], arr[child]);
		// 由于交换了节点，可能破坏了子节点的堆有序
		// 将pos设置为被交换的结点，继续下次排序
		pos = child;
	}
}
// 建堆的算法
void buildHeap(int arr[], int length) {
	// 这里从中点开始循环，从堆第一个有子节点的节点开始一直往上。注意后面的0是可取的
	for (int i = (length-1)/2; i >= 0; i--) {
		heapify(arr, length, i);
	}
}
```
归并排序
---------------
```c
void merge(int* arr, int low, int mid, int high, int* tmp) {
	// i，j分别是前后两个数组的索引起始位置
	int i = low, j = mid + 1;
	// 先将数组拷贝临时数组中-这个操作安排在在排序完成后也是可以的	
	for (int k = low ; k <= high; k++) {
		tmp[k] = arr[k];
	}
	for (int k = low; k <= high; k++) {
		// 如果前面一半的数组已经被并入，剩下的就只有后面一半的数组，直接拷贝
		if (i > mid) arr[k] = tmp[j++];
		// 同上，这里代表后面的一半的数组已经被并入，剩下的只有前面一半的数组
		else if (j > high) arr[k] = tmp[i++];
		// 这里是实际排序的地方
		// 如果后面一半数组中当前元素小于前面一半数组的元素，则拷贝后面的元素
		else if (tmp[j] < tmp[i]) arr[k] = tmp[j++];
		// 同上，拷贝前面的元素
		else arr[k] = tmp[i++];
	}
}

void sort(int arr[], int low, int high, int tmp[]) {
	// 这里可以做优化
	if (low >= high) return;
	int mid = low + (high - low) / 2;
	// 排序前半段
	sort(arr, low, mid, tmp);
	// 排序后半段
	sort(arr, mid + 1, high, tmp);
	// 合并两段子数组
	merge(arr, low, mid, high, tmp);
}
```
#### **小优化** ####
在sort时加上这个判断：
```c
// 如果这个条件成立，说明前面一半的数组小于等于后面一半的数组。也就不用合并了 
if (arr[mid] <= arr[mid+1])
		return;
```
快速排序
---------------------
