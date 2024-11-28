title: js中的异步操作
author: Billy Yin
tags: []
categories: []
date: 2019-07-05 17:07:00
---
### Node.js 中读取文件

```js
var fs = require("fs")

fs.readFile('package.json', function (err, data) {
    if (err) {
        return console.error(err);
    }
    console.log(data.toString());
    fs.readFile('no.txt', function (err, data) {
        if (err) {
            return console.error(err);
        }
        console.log(data.toString());
    });
});

console.log(123)
```

回调函数嵌套造成回调地狱

### Promise

Promise 是异步编程的一种解决方案，比传统的解决方案——回调函数和事件——更合理和更强大。它由社区最早提出和实现，ES6 将其写进了语言标准，统一了用法，原生提供了Promise对象。

所谓Promise，简单说就是一个容器，里面保存着某个未来才会结束的事件（通常是一个异步操作）的结果。从语法上说，Promise 是一个对象，从它可以获取异步操作的消息。Promise 提供统一的 API，各种异步操作都可以用同样的方法进行处理。

Promise对象有以下两个特点。

（1）对象的状态不受外界影响。Promise对象代表一个异步操作，有三种状态：pending（进行中）、fulfilled（已成功）和rejected（已失败）。只有异步操作的结果，可以决定当前是哪一种状态，任何其他操作都无法改变这个状态。

（2）一旦状态改变，就不会再变，任何时候都可以得到这个结果。Promise对象的状态改变，只有两种可能：从pending变为fulfilled和从pending变为rejected。

```js
function timeout(ms) {
  return new Promise((resolve, reject) => {
    console.log(resolve,reject)
    setTimeout(resolve, ms, 'done');
  });
}

timeout(100).then((value) => {
  console.log(value);
});
```

#### Promise 的异步操作

```js
var readFile = require('fs-readfile-promise');

readFile(fileA)
.then(function (data) {
  console.log(data.toString());
})
.then(function () {
  return readFile(fileB);
})
.then(function (data) {
  console.log(data.toString());
})
.catch(function (err) {
  console.log(err);
});
```

### Generator

Generator 函数的调用方法与普通函数一样，也是在函数名后面加上一对圆括号。不同的是，调用 Generator 函数后，该函数并不执行，返回的也不是函数运行结果，而是一个指向内部状态的指针对象。

下一步，必须调用遍历器对象的next方法，使得指针移向下一个状态。也就是说，每次调用next方法，内部指针就从函数头部或上一次停下来的地方开始执行，直到遇到下一个yield表达式（或return语句）为止。换言之，Generator 函数是分段执行的，yield表达式是暂停执行的标记，而next方法可以恢复执行。

每次调用遍历器对象的next方法，就会返回一个有着value和done两个属性的对象。value属性表示当前的内部状态的值，是yield表达式后面那个表达式的值；done属性是一个布尔值，表示是否遍历结束。

```js
function* helloWorldGenerator() {
    yield 'hello';
    yield 'world';
    return 'ending';
    yield 'here';
}

var hw = helloWorldGenerator();

console.log(hw.next());
console.log(hw.next());
console.log(hw.next());
console.log(hw.next());
console.log(hw.next());

```

#### Generator 状态机

```js
var ticking = true;
var clock = function() {
  if (ticking)
    console.log('Tick!');
  else
    console.log('Tock!');
  ticking = !ticking;
}

var clock = function* () {
  while (true) {
    console.log('Tick!');
    yield;
    console.log('Tock!');
    yield;
  }
};
```

Generator 之所以可以不用外部变量保存状态，是因为它本身就包含了一个状态信息，即目前是否处于暂停态。


#### Generator 异步操作

```js
var fs = require("fs")

function* main() {
    var result = yield find("./package.json");
    console.log(result);
}

function find(file) {
    fs.readFile(file, function (err, data) {
        it.next(data.toString());
    });
}

var it = main();
it.next();

console.log(123)
```

```js

var readFile = require('fs-readfile-promise');

function* main() {
    var result = yield readFile("./package.json");
    console.log(result.toString());
    var result2 = yield readFile("./no.json");
    console.log(result2.toString());
}

var it = main();
const result = it.next();
result.value.then((d) => {
    it.next(d)
})

console.log(123)

```

### async/await

async函数其实它就是 Generator 函数的语法糖。
相比Generator，它不需要在异步事件结束之后手动调用next。

```js
var readFile = require('fs-readfile-promise');

async function fileRead(file, file2) {
    const filecontent1 = await readFile(file)
    console.log(filecontent1.toString())
    const filecontent2 = await readFile(file2)
    console.log(filecontent2.toString())
}

fileRead('./package.json', './no.txt')

```