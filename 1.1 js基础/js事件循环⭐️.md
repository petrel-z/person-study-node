js是单线程的——执行js代码的线程只有一个
但是浏览器是多线程的：
1. js引擎线程（主线程）
2. 定时器线程
3. http请求线程

多进程：
1. 渲染进程
	1. js引擎线程
	2. 定时器线程
	3. http请求线程
	4. 事件触发线程
	5. GUI线程
2. GPU进程
3. 插件进程
4. 网络进程

如果js想执行异步任务，就将该异步任务交给浏览器其他线程去执行

## 事件驱动机制
简言之，就是
由一些用户触发的事件（click事件）或者程序自动触发的事件（定时器结束），都会触发浏览器的事件驱动机制的运作，事件触发以后，浏览器会执行该事件附带的任务（回调函数）或者触发了浏览器的其他事件（例如：用户点击按钮使得一个图像的位置改变后，会触发浏览器的重新渲染事件）

微任务特征：没有明显的任务需要其他线程来处理，一般只有回调
宏任务：有明显的任务需要其他线程来处理，还有回调

## nodejs环境
node环境的事件循环有很多阶段
timer：定时器阶段
pedding callback：回调函数阶段
poll：轮询等待新的事件，如果有timer，则直接进入到timer阶段
check：setImmediate回调函数执行
close callbacks：关闭回调执行


## node和浏览器的事件循环的区别
node有很多阶段，浏览器没有
node会先将每个阶段的任务队列执行完，微任务队列执行完，才会进入下一个阶段

## nextTick、setImmediate、setTimeout
nextTick是在同步代码执行完后，在promise.then之前执行
setImmediate是在check阶段执行
setTimeout

```js
const fs = require('fs');
fs.readFile(__filename, (data) => {
  // 读文件的回调函数
  console.log('readFile')
  setTimeout(() => {
	console.log('timeout');
  }, 0);
  setImmediate(() => {
    console.log('setImmediate');
  });
});
```
打印结果是`readFile`  `setImmediate` `timeout`
因为readFile是io操作，在poll阶段执行，打印readFile，之后会检测是否有setImmediate回调需要执行，如果有，则进入check阶段执行setImmediate回调。如果没有setImmediate，则等待新任务，此时如果有timer任务，则回到timer阶段
这段代码属于有setImmediate的情况，执行完check后，进入下一阶段，直到本次循环结束
在下一次循环执行时，在timer阶段执行timeout