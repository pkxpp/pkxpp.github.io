---
layout: post
comments: true
categories: lua
---
[TOC]

# 概念
直接看官网的解释：

> Sets the environment to be used by the given function. f can be a Lua function or a number that specifies the function at that stack level: Level 1 is the function calling setfenv. setfenv returns the given function.

说明一下stack level的意思，Level 1是调用setfenv的函数，Level 2是调用setfenv函数再上一层的调用函数，依此类推。

# 使用

## 示例代码：

	local env1 = {
		-- print = print,
	}
	function testSetfenv( )
		setfenv(1, env1)
		-- print("testSetfenv")	-- Error: testSetFenv函数所在的环境设置为env1，但是env1是没有print函数的
	end
	testSetfenv()

把testSetfenv函数的环境设置为env1，所以全局_G的函数print是不存在的，会报错。

## 构建model

可以使用setfenv构建类似model的功能，^_^！

模块文件testSetfenv.lua

	-- 一个函数
	function test()
	    print("Test setfenv.")
	end
	-- 一个变量
	testvar = 666
	-- print是_G的函数，在新环境中不存在
	-- print("test ... ")


构建代码：

	local MyModel={}
	local func = loadfile("testSetfenv.lua")
	local fenv = setfenv(func, MyModel)
	fenv()
	--MyModel就是新的模块啦
	print("test = ", MyModel.test)
	print("var = ", MyModel.testvar)
	-- MyModel.test()


小结：

* fenv需要运行，说明loadfile不运行

可以看到结果：

	test = 	function: 0087D200
	var = 	666


# 总结
* 版本支持：5.1，5.2以上貌似就没有这个接口了