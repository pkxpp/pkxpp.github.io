---
layout: post
comments: true
categories: algorithm
---

[TOC]

最近在学习JavaScript，发现囫囵吞枣印象就是不深刻，自己去练习一下才能慢慢有点感觉。另外，堆排序对我们来说太耳熟而又少用的情况下，当作一次复习。

# 堆的定义

最大（最小）堆是一棵每一个节点的键值都不小于（大于）其孩子（如果存在）的键值的树。大顶堆是一棵完全二叉树，同时也是一棵最大树。小顶堆是一棵完全完全二叉树，同时也是一棵最小树。

另外，记住这两个概念，对写代码太重要了：

* 父节点和子节点的关系：看定义
* 完全二叉树：参考[2]

# 基本操作
* Build（构建堆）
* Insert(插入）
* Delete（删除：最小或者最大的那个）

# 代码实现

首先，写代码前有两个非常重要的点：

* 用一个数组就可以作为堆的存储结构，非常简单而且易操作；
* 另外同样因为是数组作为存储结构，所以父子节点之间的关系就能根据索引就轻松找到对方了。

对于JavaScript以0作为数组索引开始，关系如下：

	nLeftIndex = 2 * (nFatherIndex+1) - 1;
	nRightIndex = 2* (nFatherIndex+1);


前面提到注意两个概念，是有助于理解的：

* 因为是数组，所以父子节点的关系就不需要特殊的结构去维护了，索引之间通过计算就可以得到，省掉了很多麻烦。如果是链表结构，就会复杂很多； 
* 完全二叉树的概念可以参考[2]，要求叶子节点从左往右填满，才能开始填充下一层，这就保证了不需要对数组整体进行大片的移动。这也是随机存储结构（数组）的短板：删除一个元素之后，整体往前移是比较费时的。**这个特性也导致堆在删除元素的时候，要把最后一个叶子节点补充到树根节点的缘由**

代码实现：

	/******************************************************
	* file    : 堆
	* author  : "page"
	* time    : "2016/11/02"
	*******************************************************/
	function Heap()
	{
		this.data = [];
	}
	
	Heap.prototype.print = function () {
		console.log("Heap: " + this.data);
	}
	
	Heap.prototype.build = function(data){
		// 初始化
		this.data = [];
		if (!data instanceof Array)
			return false;
	
		// 入堆
		for (var i = 0; i < data.length; ++i) {
			this.insert(data[i]);
		}
	
		return true;
	}
	
	Heap.prototype.insert = function( nValue ){
		if (!this.data instanceof Array) {
			this.data = [];
		}
	
		this.data.push(nValue);
		// 更新新节点
		var nIndex = this.data.length-1;
		var nFatherIndex = Math.floor((nIndex-1)/2);
		while (nFatherIndex > 0){
			if (this.data[nIndex] < this.data[nFatherIndex]) {
				var temp = this.data[nIndex];
				this.data[nIndex] = this.data[nFatherIndex];
				this.data[nFatherIndex] = temp;
			}
	
			nIndex = nFatherIndex;
			nFatherIndex = Math.floor((nIndex-1)/2);
		}
	}
	
	Heap.prototype.delete = function(  ){
		if (!this.data instanceof Array) {
			return null;
		}
	
		var nIndex = 0;
		var nValue = this.data[nIndex];
		var nMaxIndex = this.data.length-1;
		// 更新新节点
		var nLeaf = this.data.pop();
		this.data[nIndex] = nLeaf;
	
		while (nIndex < nMaxIndex ){
			var nLeftIndex = 2 * (nIndex+1) - 1;
			var nRightIndex = 2 * (nIndex+1);
	
			// 找最小的一个子节点(nLeftIndex < nRightIndex)
			var nSelectIndex = nLeftIndex;
			if (nRightIndex < nMaxIndex) {
				nSelectIndex = (this.data[nLeftIndex] > this.data[nRightIndex]) ? nRightIndex : nLeftIndex;
			}
	
			if (nSelectIndex < nMaxIndex && this.data[nIndex] > this.data[nSelectIndex] ){
				var temp = this.data[nIndex];
				this.data[nIndex] = this.data[nSelectIndex];
				this.data[nSelectIndex] = temp;
			}
	
			nIndex = nSelectIndex;
		}
	
		return nValue;
	}
	// test
	var heap = new Heap();
	heap.build([1, 3, 5, 11, 4, 6, 7, 12, 15, 10, 9, 8]);
	heap.print();
	// insert
	heap.insert(2);
	heap.print();
	// delete
	heap.delete();
	heap.print();

## 关于JavaScript的几点小结
* 这里是采用面向对象的一种实现方法，感觉上不是太优雅，不知道还有没有更好的表示方法和写法；
* 学习了数组的几个用法：push和pop的操作太好用了；
* 判断数组的方式也是临时从网上搜的（instanceof），印象不深刻，不用的话下次估计还是有可能忘掉。

# 总结

1. JavaScript的数组实现了push和pop这些操作，许多其他语言也提供了类似的数据结构和操作（比如C++的Vector），同时也支持随机操作。所以，我开始想如果这些结构上简单的加上自动排序的概念，那么一个堆就轻松搞定了，后面看到C++ STL的make_heap就知道自己知道的太少了，但也庆幸自己思维方式是对的（^_^）。JavaScript的没有去查，我想有或者实现起来很容易；
2. 自己去实现了之后，发现这个结构也很简单，只要你肯去跟它亲密接触一次就可以了；
3. JavaScript的细节部分还是不太了解，比如数组的应用上还要再翻资料才能用；对于JavaScript的灵魂还是没有接触到，精髓部分需要不断的学习和练习；
4. 这些代码，只要你去了解了概念，了解了编程的基础，就可以写的出来。但是，代码还可以写的更简洁，比如delete函数求最小的子节点的时候，左右节点的索引就不需要比较，肯定是左边的小。代码部分感觉还是可以继续优化和精简的。

# 参考
[1]《数据结构和算法分析：C语言描述》

[2][图解数据结构（8）——二叉堆 ](http://www.cnblogs.com/yc_sunniwell/archive/2010/06/28/1766751.html)

[3][数据结构：堆](http://blog.csdn.net/wypblog/article/details/8076324)