---
layout: post
comments: true
categories: node
tags: node zmq
---

[TOC]

最近在搭nodejs服务器，用到了zmq，因为用的是JavaScript，所以用的是zeromq.js[1]。（没用zmq，是因为windows下面安装zmq，node-gyp没有编译过，因为要快速出来，所以就先放弃了。）结果一开始就遇到一个问题就是，router发消息给dealer，接收收不到





# 原因
参考[2]，主要是要了解router接收消息的时候，默认第一个参数是origin socket，而发送消息的时候第一个参数要传你要发给哪一个dealer
> A Router can be used to extend request/reply sockets. When receiving messages a Router shall prepend a message part containing the routing id of the originating peer to the message. Messages received are fair-queued from among all connected peers. When sending messages, the first part of the message is removed and used to determine the routing id of the peer the message should be routed to.

测试代码：
* router.js文件

```
	const zmq = require("zeromq")
	
	async function run() {
	  const sock = new zmq.Router
	  await sock.bind("tcp://127.0.0.1:3000")
	  console.log("Worker connected to port 3000")
	
	 while (true) {
	    await sock.send("some work")
	
	 const [id, msg] = await sock.receive()
	 console.log('msg = ', id, msg.toString())
	
	 sock.send([id, "I,m broker."])
	  }
	}

	run()
```

* dealer.js

```
	const zmq = require("zeromq")

	async function run() {
	  await sock.connect("tcp://127.0.0.1:3000")
	  console.log("Producer bound to port 3000")
	
	  while (true) {
	    await sock.send("some work")
		
	 const [msg] = await sock.receive()
	 console.log('msg = ', msg.toString())
	   
	    await new Promise(resolve => setTimeout(resolve, 500))
	  }
	}

	run()
```



# 参考
[1][zmq-github](https://github.com/zeromq/zeromq.js)

[2][zeromq.js-router](http://zeromq.github.io/zeromq.js/classes/router.html)