---
layout: post
comments: true
categories: node
tags: node zmq
---

[TOC]

因为最近在弄一个nodejs的服务端，结果用的zeromq实现的rpc。结果发现rpc会出现卡住的情况，打印消息之后主要是zeromq消息丢了，发送之后，接收端没有收到。然后发现示例代码中用的是async/await这种异步调用，不太明白，所以找了些资料学习了一下，做一下笔记






# 问题
1. zeromq中为什么用异步方式？
2. async函数返回值怎么得到？
这是一开始的代码：

```
function sendMsg(server, msg, uniqueId) {
 if (!this.socket) {
  return null;
 }

 var rid;
 if (uniqueId === undefined) {
  rid = uuid.v4() + '_' + this.id;
 } else {
  rid = uniqueId;
 }

 this.socket.send([MDP.REQUEST, server, rid, JSON.stringify(msg)])
 return rid;
}
```

# 理解
总的来说呢，Promise和async/await都属于异步调用的方式，并且能解决‘地域回调’的问题。而且async/await写起来更符合同步代码的方式，阅读起来更好理解，参考[1]
## Promise
Promise一开始也是为了解决回调问题的，参考[2][3]
> Promise 是一个对象，它代表了一个异步操作的最终完成或者失败

简单来讲就是，异步调用之后会有成功和失败的结果。你可以在异步调用完之后统一处理，不需要在各个分支回调处理。即不需要把回调传进去，而是最终对一个Promise对象处理。怎么处理？时间点就是Promise.then函数。

```
const promise1 = new Promise(function(resolve, reject) {
  setTimeout(function() {
    resolve('foo');
  }, 300);
});

promise1.then(function(value) {
  console.log(value);
  // expected output: "foo"
});

console.log(promise1);
// expected output: [object Promise]
```

另外，注意then的参数是一个函数，很多时候看到的是箭头函数[4]，所以一开始接触会觉得有点奇怪。

## async
async很好理解，就是异步函数。参考[5]

需要注意的是它的返回值是Pomise对象，所以我一开始的sendMsg函数写成下面之后的形式，同步调用sendMsg得到的返回值rid就是不对的。这个开始也碰到问题了

```
async function sendMsg(server, msg, uniqueId) {
 // 省略
 await this.socket.send([MDP.REQUEST, server, rid, JSON.stringify(msg)])
 return rid;
}
```
另外一个是async中可以用await，中断async函数。我理解是这个时候yield丢回去的就是一个Promise对象，即使你await立马返回值本身也是一个Promise对象
```
async function test1(){
 var a = await 3;
 return a;
}
console.log(test1())
console.log(test1().then((resolve, reject)=>{
 console.log('resolve:', resolve)
}))
//结果
Promise { <pending> }
Promise { <pending> }
resolve: 3
```
## await
await大概有以下几点，参考[6]：

* 只能在async函数中用
* await 表达式会暂停当前 async function 的执行，等待 Promise 处理完成
* 因为是异步调用，async这个时候如果被await中断继续往下执行，但是不会阻塞停在这里。返回的是Promise对象，只是这个Promise对象还没有处理完，所以返回的Promise.then函数是还没有开始执行的。

拿参考[5]中的例子

```
var resolveAfter2Seconds = function() {
  console.log("starting slow promise");
  return new Promise(resolve => {
    setTimeout(function() {
      resolve("slow");
      console.log("slow promise is done");
    }, 2000);
  });
};

var resolveAfter1Second = function() {
  console.log("starting fast promise");
  return new Promise(resolve => {
    setTimeout(function() {
      resolve("fast");
      console.log("fast promise is done");
    }, 1000);
  });
};

var concurrentStart = async function() {
  console.log('==CONCURRENT START with await==');
  const slow = resolveAfter2Seconds(); // starts timer immediately
  const fast = resolveAfter1Second(); // starts timer immediately
  // 1. Execution gets here almost instantly
  console.log(await slow); // 2. this runs 2 seconds after 1. 
  console.log(await fast); // 3. this runs 2 seconds after 1., immediately after 2., since fast is already resolved
}
```

结果：

```
==CONCURRENT START with await==
starting slow promise
starting fast promise
fast promise is done
slow promise is done
slow
fast
```

(1)两个函数resolveAfter2Seconds和resolveAfter1Second没有用await，就在同一帧执行了，所以前三条打印马上就出来了

(2)第四条打印fast promise is done会在第1秒结束的时候打印，因为resolveAfter1Second函数执行完了，并且Promise处理完了

(3)最后三条打印一起打印出来了，主要是fast要最后打印，因为await slow阻塞了async最后一个console.log的执行。即使fast的Promise已经处理了

# 解决和总结
把异步函数抽出来另外一个函数，就不影响sendMsg的返回值了

```
async function send(msg) {
    if (!this.socket) {
        return;
    }
    try {
        await this.socket.send(msg);
    }catch(err){
        console.error('unable to send')
    }
}

function sendMsg(server, msg, uniqueId) {
 // 省略
 send.call(this, [MDP.REQUEST, server, rid, JSON.stringify(msg)]).then((resolve, reject)=>{
  console.log('send result...........', resolve, reject)
 });
 return rid;
}

```
# 参考
[1][Nodejs 异步处理的演进](https://www.jianshu.com/p/db31116e6d71)

[2][Promise](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise)

[3][Promise使用](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Using_promises)

[4][箭头函数](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Functions/Arrow_functions)

[5][async](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/async_function)

[6][await](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/await)

[7][Nodejs 中使用 Async/Await](https://juejin.im/post/5a733ab95188255efc5f24d1)

[8][async/await](https://www.zcfy.cc/article/mastering-async-await-in-node-js-risingstack)

[9][掌握 Node.js 中的 async/await](https://www.zcfy.cc/article/mastering-async-await-in-node-js-risingstack)