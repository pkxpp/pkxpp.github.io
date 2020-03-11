---
layout: post
comments: true
categories: engine
tags: engine cocos socket.io
---

[TOC]

cocos客户端要用到socket.io，发现有问题，参考了网上的文档，有些有点过时了，重新补一个





# 步骤
* 下载socket.io
**注意：是需要下载客户端**，一开始直接把socket.io的GitHub下载下来了，发现并不是这么用的
参考[3][4][5]有提到，下载socket.io.js，添加到script中

下载连接：参考[6]

最终拿到一个socket.io.js的文件
![socket.io-client](https://github.com/pkxpp/pkxpp.github.io/blob/master/_posts/img/socket.io-client.jpg?raw=true)

* 修改文件内容

参考[1]，就是包一个东西，在前面下载的socket.io.js的文件内容用下面的代码包起来

```
	 if (!cc.sys.isNative) {
	     // SocketIO 原始代码
	 }
```

* 添加到cocos creator中

这个很简单，直接拖到界面的script中之后就可以了，其实就是复制过去
![add script to cocos creator](https://github.com/pkxpp/pkxpp.github.io/blob/master/_posts/img/add%20script.jpg?raw=true)

* 设置插件
参考[5]，在资源管理器中点击socket.io.js这个文件，在属性检查其里面勾上导入插件，即可
![script snippet of cocos creator](https://github.com/pkxpp/pkxpp.github.io/blob/master/_posts/img/cocos%20creator%20script%20snippets.jpg?raw=true)

> 参考[4]中说的需要在onLoad函数中添加一个什么语句，是不需要的，估计是老的版本。最新的版本是不需要的

```
    // use this for initialization
    onLoad: function () {
        // if(!cc.sys.isNative){
        //     window.io = SocketIO
        // }
        // else{
        //     require('socket.io')
        // }
        //this.label.string = this.text;
        this.GetAvgConnServerLoadNums()
    },
```

# 使用
不需要require，直接用io这个模块就可以了

```
const socket = io.connect('http://127.0.0.1:5000');
```

# 问题
* 无法打开socket.io.js这个文件以及TypeError啥的

重新开了一个工程就好了，不知道为毛，所以如果正常操作，参考[5]中的步骤就ok了

# 参考
[1][官网：网络接口](https://docs.cocos.com/creator/manual/zh/scripting/network.html)

[2][socket.io](https://github.com/socketio/socket.io)

[3][cocos creator 1.8+socket.io （顺带express）的简单实现](https://forum.cocos.org/t/cocos-creator-1-8-socket-io-express/60017)

[4][主题 : CocosCreator + socketIO简易教程(更新至1.0)](http://www.cocoachina.com/bbs/read.php?tid-458031-fpage-2.html)

[5][CocosCreator游戏开发---菜鸟学习之路（二）SocketIO简易教程](https://www.cnblogs.com/PleaseInputEnglish/p/7919718.html)

[6][socket.io-client](https://github.com/socketio/socket.io-client)

