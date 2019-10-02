---
layout: post
comments: true
categories: lua
---

[TOC]

# 问题描述

我们的项目的lua版本升级到了5.3，为了兼容5.1的内容，我们自己写了setfenv和getfenv两个函数(之前说过5.2之后没有了setfenv和getfenv接口了[1]）。但是，在用的时候遇到坑，lua报错，但是在某个函数中打印print函数就正常了，好尴尬~。最后在同事的帮忙下，终于发现了主要原因：**当尾调用fgetenv的时候得到的并不是你想要的那个函数的环境！**





# 测试设计


	local nLevel = 1
	function f1()
	    local t = debug.getinfo(nLevel);
	    print("In f1 = ", t.func, t.what, t)
	    return f2()
	end
	
	function f2()
	    local t = debug.getinfo(nLevel);
	    print("In f2 = ", t.func, t.what, t)
	    return f3()
	    
	end
	
	function f3()
	    local t = debug.getinfo(nLevel);
	    print("In f3 = ", t.func, t.what, t)
	    return 
	end
	
	print(f1, f2, f3)
	f1()


**说明：**

* t.func会打印函数的地址，t.what是类型（看源码有C，main, Lua，tail四种：分别表示是调用C函数，load一个文件调用，调用lua函数，尾调用）
* f1尾调用f2，f2尾调用f3

# 总结
* 1.当nLevel=1的时候，也就是打印当前函数自己的环境，不论是5.1还是5.3都是正确的，没有什么好说的。

5.1打印结果：

	function: 0027C688    function: 0027C6A8    function: 0027C6C8
	In f1 =     function: 0027C688    Lua    table: 0044F698
	In f2 =     function: 0027C6A8    Lua    table: 0044F7D8
	In f3 =     function: 0027C6C8    Lua    table: 0044F850


5.3打印结果：

	function: 000000000031a490    function: 000000000031a4d0    function: 000000000031a510
	In f1 =     function: 000000000031a490    Lua    table: 000000000031a6d0
	In f2 =     function: 000000000031a4d0    Lua    table: 000000000031a790
	In f3 =     function: 000000000031a510    Lua    table: 000000000031a850`


* 2.当nLevel=2，也就是打印调用当前函数的函数的环境，就差别大了

5.1打印结果：

	function: 0023C688    function: 0023C6A8    function: 0023C6C8
	In f1 =     function: 0023C588    main    table: 0044F698
	In f2 =     nil    tail    table: 0044F800
	In f3 =     nil    tail    table: 0044F8C8

5.3打印结果：


	function: 00000000003ba490    function: 00000000003ba4d0    function: 00000000003ba510
	In f1 =     function: 00000000003beb50    main    table: 00000000003ba6d0
	In f2 =     function: 00000000003beb50    main    table: 00000000003ba810
	In f3 =     function: 00000000003beb50    main    table: 00000000003ba8d0


**说明：**

(1) 5.1中的堆栈信息三条不一样，类型为tail的堆栈信息func是空的

(2) 5.3中的函数都是一样的

(3) 这里是因为尾调用规定了重用了上一层函数的堆栈[2]

* 3.按照同事的解释：

(1)5.1下面尾调用会在堆栈里面插入一条类型为tail的信息，看5.1打印结果可以看到，但是根本没有真正的函数。*lua实现的尾调用可以无限调用，不受堆栈上限的限制*。

(2)5.3已经改成了_ENV的概念，也就是任何一个函数（一个文件当作一个函数）都有自己的_ENV。所以，5.3结果都是main类型，也就是这三个函数所在文件的一个function

* 4.lua5.1中的尾调用会占用一条堆栈信息，而5.3中不会。可以把nLevel设置为3，可以看到5.1这个时候打印的f2的第三层是main类型。而在5.3中的nLevel=2就是main类型的调用了

* 最后简单解释一下，加了一条打印就正常的原因。因为一个函数默认的_ENV是空的，当加了一条pirnt函数，会生成_ENV，因为你用到了print函数。而我们自己写的getfenv是根据_ENV字段来的，所以有_ENV和没有_ENV的区别，后面就导致报错了。至于_ENV这个字段为毛有时候有，有时候没有就自己搜索看看了^_^

# 参考
[1][Lua学习笔记(7)setfenv](http://pkxpp.github.io/2016/08/17/lua%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0(7)setfenv/)

[2][3.4.10 – Function Calls](http://www.lua.org/manual/5.3/manual.html#3.3.6)