title: React 虚拟DOM的 $$typeof 属性
author: Billy Yin
tags: []
categories: []
date: 2019-11-15 11:22:00
---
研究React虚拟DOM时尝试打印JSX元素，有个属性不太熟悉
![capture.png](https://upload-images.jianshu.io/upload_images/9594241-5036feed28f45e22.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

其它的都可以理解，但是这个 $$typeof 是啥？？？

在研究了网上的资料后，得出结论，这是用于防止XSS攻击的一种方式。

### JSX中插入HTML结构

在传统前端开发中经常使用HTML字符串拼接然后append的方式向页面中引入HTML结构

但这样的写法在React中好像不可以

```
const html = '<div>123</div>'

function App(props) {
    return (
        <div className="App">
            {html}
        </div>
    );
}
```

![image.png](https://upload-images.jianshu.io/upload_images/9594241-415b5ee4d6c4489e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

原因：React等现代前端lib为了防止XSS攻击，对 < > 等符号做了一层转义，称为 `escape `

如果真的想插入HTML结构而非文本呢？ 

### dangerouslySetInnerHTML

>dangerouslySetInnerHTML 是 React 为浏览器 DOM 提供 innerHTML 的替换方案。

```
const html = '<div>123</div>'

function App(props) {
    return (
        <div
            className="App"
            dangerouslySetInnerHTML={{ __html: html }}
        >
        </div>
    );
}
```

![image.png](https://upload-images.jianshu.io/upload_images/9594241-8411dad579087a4f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 风险？

场景：你的服务器存在一个漏洞，使得用户可以存储任意的JSON对象，但客户端却需要一个 string 类型的值。

```
// 期望该字段为text但是拿到了JSON
let expectedTextButGotJSON = {
  type: 'div',
  props: {
    dangerouslySetInnerHTML: {
      __html: '<img src onerror="alert(document.cookie)"/>'
    },
  },
  // ...
};

// JSX
<p>
  {expectedTextButGotJSON}
</p>
```

按照Virtual DOM 的渲染机制（[学习React VirtualDom](https://www.jianshu.com/p/536d43ce8479)），有风险的节点会直接被渲染到DOM树。

### $$typeof

React  0.14版本之后，在渲染节点时，新增了一步校验

首先新增一个常量
```
var REACT_ELEMENT_TYPE =
  (typeof Symbol === 'function' && Symbol.for && Symbol.for('react.element')) ||
  0xeac7;

```
如果浏览器支持 Symbol：

生成虚拟DOM对象时，自带属性$$typeof :  Symbol.for('react.element')

React渲染时，会做校验：
```
ReactElement.isValidElement = function (object) {
  return typeof object === 'object' && object !== null && object.$$typeof === REACT_ELEMENT_TYPE;
};
```

为什么用 Symbol.for 呢？

1 Symbol属性不可存放于JSON中

2 Symbol('react.element') !== Symbol('react.element') 但 Symbol.for('react.element') === Symbol.for('react.element')