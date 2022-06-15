---
layout: post
comments: true
categories: lua
---


自己项目用到协程，但是里面报错的话，会导致协程撕掉。导致后面刷的log不是很直观，所以得想个办法处理一下






# 描述
* 环境：lua 5.1

## 尝试
自然的就想到用pcall/xpcall来执行了，大概代码如下：

```lua
function LItemMgr:_CreateCoroutineTask(fnTask, ...)
    if type(fnTask) ~= "function" then
        return;
    end

    local co = coroutine.create(function (...)
		local tbParams = {...};
		while true do
		    local status , bFinished = xpcall(function ()
		        return fnTask(unpack(tbParams));
		    end, debug.traceback); 
                   tbParams = {coroutine.yield(bFinished)};
		end
	end)

    return co;
end
```
就是把我想要在协程里面执行的**Function**加一个保护壳运行，结果函数内不报错的情况下就出错了

```lua
attempt to yield across metamethod/C-call boundary
```


首次看到这个，很纳闷，协程里面yield和C代码有啥关系？因为我这里并没有我导出的C++接口到lua



# 原因
参考\[1\]里面解释的很清楚了

```cpp
Internally, Lua uses the C longjmp facility to yield a coroutine. Therefore, if a function foo calls an API function and this API function yields (directly or indirectly by calling another function that yields), Lua cannot return to foo any more, because the longjmp removes its frame from the C stack.
```


简单来说，等于是没办恢复c的栈了，就报这个错误了。对于我的情况如下：

* 传进来的fnTask这个函数里面有yield，因为我打算分帧执行某些操作
* 但是，xpcall保护运行fnTask的时候，等于是我调用了一个C函数（xpcall等于是API，C语言实现的）
* 那么，我的情况等于是从C函数调用lua（fnTask），而这个lua函数里面有yield操作
* 所以，yield的时候，等于是把C的堆栈给干掉了。等下次resume的时候，恢复不了堆栈所以抛出错误了



# 解决
参考\[3\]



## 方法一：试一下补丁
参考\[4\]

我是用的vs编的lib，大概步骤：

* 1.把lua-5.1.5-coco-1.1.9.patch下载下来
* 2.因为这个patch需要目录是lua-5.1.5，我直接从参考\[4\]里面另外下载了一个lua-5.1.5
* 3.右键这个文件->git ->review/apply single patch...，全部应用
* 4.会得到一个lua-5.1.5-coco文件夹和源码lua-5.1.5同级
* 5.把lua-5.1.5-coco/src的全部代码覆盖到lua-5.1.5/src中，这里其实多了连个文件：lcoco.h和lcoco.c
* 6.vs新建一个工程，把lua的源码添加进去，包括上面两个多的文件
* 7.这样新编译出来的lib和dll替代lua-5.1.5源码的lib和dll
* 8.测试相同的代码，不报错了，大功告成！



## 方法二：避免yield即可
我仔细想了一下，只要自己xpcall里面不要有yield即可。那么简单来说就是，我在fnTask里面保护运行部分代码（xpcall），只要xpcall里面没有yield就行了。

```lua
function LItemMgr:TickBlockByCoroutine(nIndex, nStart, bFrame)
    local nCount = 0;
    local fnToDo = function ()
        local nBlockID, uuid;
        local tbItemInfo = self.ItemList[nIndex];
        if tbItemInfo then
            nBlockID, uuid = unpack(tbItemInfo);
            nCount = nCount + 1;
        end
        
        print("Real Do Something: ", nIndex, nBlockID, uuid)
        -- a.b = 5;
    end
    -- fnToDo();
    local bRet, szErr = xpcall(fnToDo, debug.traceback)
    print("xpcall invoke result: ", bRet, szErr);
    
    local COUNT_PER_FRAME = 2;
    if bFrame and (nStart + nCount) % COUNT_PER_FRAME == 0 then
        _, bFrame = coroutine.yield(false)
        print("bNewFrame = ", bFrame)
    end

	return nCount;
end
```
简单的一个实现如下，把真正要做的代码（fnToDo）用xpcall包一下。其他代码其实赢不会错什么错，而且也是协程分帧必备代码，就可以省一点了。



**小结：**

* 可行。
* 其实，只要把核心代码，或者觉得明显有错误的代码用xpcall保护运行即可
* 缺点：这样的代码设计很隐晦，其他人很容易改崩掉，所以得加好注释！



# 参考
\[1\]\[4.7 – Handling Yields in C\]([http://www.lua.org/manual/5.2/manual.html#4.7](http://www.lua.org/manual/5.2/manual.html#4.7))

\[2\]\[如何在Lua5.1.4中实现这样的效果\]([http://bbs.chinaunix.net/forum.php?mod=viewthread&action=printable&tid=4065715](http://bbs.chinaunix.net/forum.php?mod=viewthread&action=printable&tid=4065715))

\[3\]\[**attempt\_ *****to***** *****yield***** *****across***** *****metamethod*****/*****C-call***** \_boundary**\]([https://www.cnblogs.com/lightsong/p/5785615.html](https://www.cnblogs.com/lightsong/p/5785615.html))

\[4\]\[Coco — True C Coroutines for Lua\]([http://coco.luajit.org/](http://coco.luajit.org/))