title: 使用 Node.js 模拟浏览器文件上传
author: Billy Yin
tags: []
categories: []
date: 2021-03-09 23:34:00
---
### 第一条线: FormData 相关
浏览器上传文件依赖于[FormData](https://developer.mozilla.org/zh-CN/docs/Web/API/FormData)，一般使用：
```js
let fd = new FormData()
let request = new XMLHttpRequest();
// file 从 <input type="file">  表单项得到
fd.append('file', file)
request.open("POST", "submitform.php");
request.send(formData);
```
那么这个file是什么格式呢？翻看 [FormData 对象的使用](https://developer.mozilla.org/zh-CN/docs/Web/API/FormData/Using_FormData_Objects)

> FormData 对象的字段类型可以是 Blob, File, 或者 string: 如果它的字段类型不是Blob也不是File，则会被转换成字符串类

Blob和File分别是什么类型呢？继续翻看文档 
* [File](https://developer.mozilla.org/zh-CN/docs/Web/API/File)
文件（File）接口提供有关文件的信息，并允许网页中的 JavaScript 访问其内容。`File`接口基于`Blob`，继承了 blob 的功能并将其扩展使其支持用户系统上的文件。
``var myFile = new File(bits, name[, options])``，其中bits的类型是ArrayBuffer
* [Blob](https://developer.mozilla.org/zh-CN/docs/Web/API/Blob)
Blob 对象表示一个不可变、原始数据的类文件对象。
``var aBlob = new Blob( array, options )``，array 是一个由ArrayBuffer, ArrayBufferView, Blob, DOMString 等对象构成的 Array ，或者其他类似对象的混合体，它将会被放进 Blob。

看来只要能获取到ArrayBuffer类型的对象，就能构建File或Blob实例，也就能完成文件上传了。

### 第二条线：Node.js 相关
首先 Node.js 并不原生支持 XMLHttpRequest ，可以使用 https://www.npmjs.com/package/xmlhttprequest 代替
Node.js 中文件相关模块是 [fs](http://nodejs.cn/api/fs.html)
```
const fs = require('fs')
```
Node.js 13 版本后，对ES Module也支持了：
```
import fs from 'fs'
```
这里我们用 [fs.readfilesync](http://nodejs.cn/api/fs.html#fs_fs_readfilesync_path_options) 同步读取文件
```
const file = fs.readfilesync('./README.md')
```
此时的file是个[Buffer](http://nodejs.cn/api/buffer.html)对象，要怎么转成 [ArrayBuffer](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/ArrayBuffer) 呢？

这里提供一种方法
```
function toArrayBuffer(buf:Buffer): ArrayBuffer {
  var ab = new ArrayBuffer(buf.length);
  var view = new Uint8Array(ab);
  for (var i = 0; i < buf.length; ++i) {
      view[i] = buf[i];
  }
  return ab;
}

```
然后
```
  const file = new Blob([arrayBuffer])
```

PS : stackoverflow 上有好些回答表示使用 ``fs.createReadStream`` 返回的可读流也可以append到FormData中，但个人实验下来实际上得到的文件是错乱的。

最后，把第二条线和第一条线串起来就可以完成 Node.js 模拟浏览器进行文件上传了。