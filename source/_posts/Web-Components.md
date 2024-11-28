title: Web Components
author: Billy Yin
tags: []
categories: []
date: 2020-06-07 07:40:00
---

>Web Components允许创建**可重用**的**定制元素**，并且在web应用中**使用**它们。

### HTML Templates


##### Template
[`<template>`](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/template)

HTML内容模板（`<template>`）元素是一种用于保存客户端内容机制，该内容在加载页面时不会呈现，但随后可以在运行时使用JavaScript实例化。
##### [<slot>](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/slot)

作为 Web Components 技术套件的一部分，是Web组件内的一个占位符。

### Custom Elements
##### customElements.define()
注册一个 custom element，该方法接受以下参数：
*  `必填参数`，表示所创建的元素名称的符合 [`DOMString`](https://developer.mozilla.org/zh-CN/docs/Web/API/DOMString "DOMString 是一个UTF-16字符串。由于JavaScript已经使用了这样的字符串，所以DOMString 直接映射到 一个String。") 标准的字符串。注意，custom element 的名称不能是单个单词，且其中[必须要有短横线](https://stackoverflow.com/questions/22545621/do-custom-elements-require-a-dash-in-their-name)。
*    `必填参数`，用于定义元素行为的 [类](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes) 。
*   `可选参数`，一个包含 `extends` 属性的配置对象，是可选参数。它指定了所创建的元素继承自哪个内置元素，可以继承任何内置元素。

##### 生命周期
connectedCallback：当 custom element首次被插入文档DOM时，被调用。
disconnectedCallback：当 custom element从文档DOM中删除时，被调用。
adoptedCallback：当 custom element被移动到新的文档时，被调用。
attributeChangedCallback: 当 custom element增加、删除、修改自身属性时，被调用。

### Shadow DOM
* 开启方法： Chrome -> Devtool -> Settings -> Preferences -> Show user agent shadow DOM
* 观察 `<input>` `<audio>` `<video>` `<select>` 等标签
>选择器无法越过 shadow 边界 

![image.png](https://upload-images.jianshu.io/upload_images/9594241-e19291dc1f692bbf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1024)

[https://caniuse.com/#feat=shadowdomv1](https://caniuse.com/#feat=shadowdomv1)

> **`Element.attachShadow()`** 方法给指定的元素挂载一个Shadow DOM，并且返回对 [ShadowRoot](https://developer.mozilla.org/zh-CN/docs/Web/API/ShadowRoot) 的引用。


### 示例
[https://github.com/mdn/web-components-examples](https://github.com/mdn/web-components-examples)

### 相关框架/库
[https://stenciljs.com/docs/introduction](https://stenciljs.com/docs/introduction)

[https://www.polymer-project.org/](https://www.polymer-project.org/)

