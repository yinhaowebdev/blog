title: event loop
author: Billy Yin
tags: []
categories: []
date: 2019-03-11 23:24:00
---
下面一段简单的代码

```js
console.log('script start');
setTimeout(function() {
  console.log('setTimeout');
}, 0);
console.log('script end');
```
执行结果是
```bash
script start
script end
setTimeout
```
为什么定时器即使设置0秒，也会在最后一行代码后执行呢？

### 执行栈和任务队列

JS 的执行是单线程的

JS 分为同步任务和异步任务

JS 执行栈是JS代码运行的栈，主线程会在其上依次执行代码

异步任务被推入任务队列

当JS执行栈空闲时，主线程读取任务队列，并依次执行它们的回调函数

![事件循环.png](https://upload-images.jianshu.io/upload_images/9594241-54a8d2c11c23a698.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

JS 执行栈 是先进后出， 任务队列 是先进先出。

不难理解，以上代码片段中，定时器回调函数没有在第一时间执行，而是等到第一次执行栈为空之后再去执行的，所以产生了输出的结果

### MacroTask 和 MicroTask  

想一想下面代码的执行结果，看起来没那么简单了...

```js
console.log('script start');
setTimeout(function() {
  console.log('setTimeout1');
}, 0);
new Promise((resolve)=>{
  console.log('promise1')
  setTimeout(function() {
    console.log('setTimeout2');
  }, 0);
  resolve()
  console.log('promise2')
}).then(()=>{
  setTimeout(function() {
    console.log('setTimeout3');
  }, 0);
  console.log('then')
})
setTimeout(function() {
  console.log('setTimeout4');
}, 0);
console.log('script end');
```
执行结果是

```shell
script start
promise1
promise2
script end
then
setTimeout1
setTimeout2
setTimeout4
setTimeout3
```

事件队列中的事件分为两类

MacroTask (tasks): setTimeout, setInterval, setImmediate, Ajax, onClick(), OnLoad()

MicroTask (jobs): [process.nextTick](https://nodejs.org/zh-cn/docs/guides/event-loop-timers-and-nexttick/), Promises, [MutationObserver](https://developer.mozilla.org/zh-CN/docs/Web/API/MutationObserver)

![tasks and jobs.jpg](https://upload-images.jianshu.io/upload_images/9594241-e9d76494184b9f17.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

流程的细节是：

1 把最早的任务(task A)放入任务队列
2 如果 task A 为null (那任务队列就是空)，直接跳到第6步
3 将 currently running task 设置为 task A
4 执行 task A (也就是执行回调函数)
5 将 currently running task 设置为 null 并移出 task A
6 执行 microtask 队列
    a: 在 microtask 中选出最早的任务 task X
    b: 如果 task X 为null (那 microtask 队列就是空)，直接跳到 g
    c: 将 currently running task 设置为 task X
    d: 执行 task X
    e: 将 currently running task 设置为 null 并移出 task X
    f: 在 microtask 中选出最早的任务 , 跳到 b
    g: 结束 microtask 队列
7 跳到第一步

所以，每次执行一个macrotask就会把当前队列里所有的microtask执行完，某种角度上看，microtask似乎比macrotask具有更高的优先级。

还有一点要注意的，按照以上循环规则，每执行一次macrotask就会触发一次UI渲染检查，所以推荐把会带来UI改动的事件尽量放在microtask中。

### 关于setTimeout的一个注意事项

```js
console.log('script start');
setTimeout(function() {
  console.log('5');
}, 5);
setTimeout(function() {
  console.log('4');
}, 4);
setTimeout(function() {
  console.log('3');
}, 3);
setTimeout(function() {
  console.log('2');
}, 2);
setTimeout(function() {
  console.log('1');
}, 1);
setTimeout(function() {
  console.log('0');
}, 0);
console.log('script end');
```
执行结果？
```bash

```