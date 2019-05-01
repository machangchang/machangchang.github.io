---
layout: post
title: 二分查找—十年BUG
categories: Java
description: BinarySearch
keywords: BinarySearch
---

二分查找—十年BUG

偶然发现了一篇文章，介绍了关于 Java 库里的一个bug，十分有意思。

Java库里面的二分查找，有一个埋藏了10年之久的bug。这个bug呢，在 java.util.Arrays.binarySearch 里面，虽然这个bug的修复也已经是10年前的事了。那么我们来看下当年的错误代码吧。

![](/images/blog/2018-03-29-BinarySearch/BinarySearch_001.jpg)

大家可能很难看出来，那毕竟这个bug藏了10年，不太容易发现。问题就在于
`int mid = (low + high) / 2;`
这里，`low + high`是会溢出的。
正确的解法是：`int mid = (low + high) >>> 1; `

正如修复bug的同学说的那样：
"Can't even compute average of two ints" is pretty embarrassing.

那正确的二分查找如何写呢？

首先是最普通的二分查找。

```java
public static int search(int[] arr, int target) {
		
		if (arr == null || arr.length == 0){
			return -1;
		}
		
		int low = 0;
		int heigh = arr.length - 1;
		int mid;
		while (low <= heigh) {
			mid = (low + heigh)>>1;
			if (arr[mid] > target){
				heigh = mid - 1;
			} else if (arr[mid] < target){
				low  = mid + 1;
			} else {
				return mid;
			}
		}
		return -1;
	}
```

如果要求是查找某个数第一次出现的位置，则该这样写：

```java
public static int searchFirst(int[] arr, int target) {
		
		if (arr == null || arr.length == 0){
			return -1;
		}
		
		int low = 0;
		int heigh = arr.length - 1;
		int mid;
		while (low <  heigh) {
			mid = (low + heigh)>>1;
			if (arr[mid] > target){
				heigh = mid - 1;
			} else if (arr[mid] < target){
				low  = mid + 1;
			} else {
				heigh = mid;
			}
		}
		if (arr[low] == target){
			return low;
		}
		return -1;
	}
```

> 声明：本站采用开放的[知识共享署名-非商业性使用-相同方式共享 许可协议](https://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh)进行许可。
