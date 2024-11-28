title: CommonJS & ES6 Module
author: Billy Yin
tags: []
categories: []
date: 2020-05-14 23:59:00
---
### 原始
最初，我们需要写互相依赖的功能时：
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <div id="root"></div>
    <script>
        function reduce(arr, func) {
            return arr.reduce((a, b) => func.call(this, a, b))
        }
        function add(a, b) {
            return a + b;
        }
        function sum(arr) {
            return reduce(arr, add);
        }
        var values = [1, 2, 4, 5, 6, 7, 8, 9];
        var result = sum(values)
        document.getElementById("root").innerHTML = result;
    </script>
</body>
</html>
```
使用script：

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <div id="root"></div>
    <script type="text/javascript" src="./reduce.js"></script>
    <script type="text/javascript" src="./add.js"></script>
    <script type="text/javascript" src="./sum.js"></script>
    <script type="text/javascript" src="./main.js"></script>
</body>
</html>
```
### IIFE

```
var myApp = {};


(function () {
    myApp.reduce = function (arr, func) {
        return arr.reduce((a, b) => func.call(this, a, b))
    }
})()

(function () {
    myApp.add = function (a, b) {
        return a + b;
    }
})()

(function () {
    myApp.sum = function (arr) {
        return myApp.reduce(arr, myApp.add);
    }
})()

(function () {
    var values = [1, 2, 4, 5, 6, 7, 8, 9];
    var result = myApp.sum(values)
    document.getElementById("root").innerHTML = result;
})()
```
eg: JQuery

### CommonJS
CommonJS 不是一个 JavaScript 库。它是一个标准化组织。
在CommonJS中，文件=>模块

```
var sum = require('./sum')
var values = [1, 2, 4, 5, 6, 7, 8, 9];
var result = sum(values)
document.getElementById("root").innerHTML = result;
```

```
module.exports = function (arr) {
    // ...
}
```

· require是同步的
· require第一次加载模块会执行文件内容，之后加载模块只会加载上次导出的值的拷贝
```
//file2.js
console.log(12)

let obj = {
    a: 1,
}
// setTimeout(() => {
//     obj.a = 2
// }, 10)

setTimeout(() => {
    obj = { a: 2 }
}, 10)

module.exports = obj
```

```
//file1.js
const file2 = require('./file2')
console.log(file2)

setTimeout(() => {
    const file2 = require('./file2')
    console.log(file2)
}, 1000)

```

PS: <script>标签的module类型需要在服务环境下运行（跨域校验）

### ES6 Module
```
import file from './file2.mjs';
console.log(file)
```

```
console.log(12)
let obj = {
    a: 1,
}
setTimeout(() => {
    obj = { a: 2 }
}, 10)
// module.exports = obj
export default obj
```
