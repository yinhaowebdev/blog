title: React.PureComponent & React.memo()
author: Billy Yin
tags: []
categories: []
date: 2019-08-15 15:24:00
---
>问题： react中如何尽量减少不必要的渲染

### shouldComponentUpdate()

它是react的一个生命周期

![图片来自：https://segmentfault.com/a/1190000016494335](https://upload-images.jianshu.io/upload_images/9594241-39958e7a1941ea99.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

组件state或props被更新时可以通过这个生命周期判断是否继续渲染。

它接受两个参数`nextProps`、`nextState`，返回一个布尔值。

若不在代码中声明该生命周期，react默认的处理是：

```js
  shouldComponentUpdate(nextProps, nextState) {
    return true;
  }
```

编写代码的时候可以通过该生命周期来优化渲染

```
    shouldComponentUpdate(nextProps, nextState) {
        if (this.props.name === nextProps.name) {
            return false;
        }
        return true;
    }
```

### React.PureComponent
大部分情况下，你可以使用 `React.PureComponent` 来代替手写 `shouldComponentUpdate`。

以下两者效果相同：
```
import React from 'react';

class Child extends React.Component {
    shouldComponentUpdate(nextProps, nextState) {
        if (this.props.name === nextProps.name) {
            return false;
        }
        return true;
    }
    render() {
        console.log('render')
        return (
            <div>
                child,{this.props.name}
            </div>
        );
    }
}

export default Child;

```

```
import React from 'react';

class Child extends React.PureComponent {
    render() {
        console.log('render')
        return (
            <div>
                child,{this.props.name}
            </div>
        );
    }
}

export default Child;
```
`React.PureComponent`只进行浅比较（shallowEqual）：

[react shallowEqual 源码](https://github.com/facebook/react/blob/a9b035b0c2b8235405835beca0c4db2cc37f18d0/packages/shared/shallowEqual.js "packages/shared/shallowEqual.js")
```
const hasOwn = Object.prototype.hasOwnProperty

function is(x, y) {
  if (x === y) {
    return x !== 0 || y !== 0 || 1 / x === 1 / y
  } else {
    return x !== x && y !== y
  }
}

export default function shallowEqual(objA, objB) {
  if (is(objA, objB)) return true

  if (typeof objA !== 'object' || objA === null ||
      typeof objB !== 'object' || objB === null) {
    return false
  }

  const keysA = Object.keys(objA)
  const keysB = Object.keys(objB)

  if (keysA.length !== keysB.length) return false

  for (let i = 0; i < keysA.length; i++) {
    if (!hasOwn.call(objB, keysA[i]) ||
        !is(objA[keysA[i]], objB[keysA[i]])) {
      return false
    }
  }

  return true
}
```

### React.memo()

它是React 16.8 的新特性之一

`React.memo` 为高阶组件。它与 `React.PureComponent` 非常相似，但它适用于函数组件，但不适用于 class 组件。

使用方法：

```js
function MyComponent(props) {
  /* 使用 props 渲染 */
}
function areEqual(prevProps, nextProps) {
  /*
  如果把 nextProps 传入 render 方法的返回结果与
  将 prevProps 传入 render 方法的返回结果一致则返回 true，
  否则返回 false
  */
}
export default React.memo(MyComponent, areEqual);
```

参考：[https://www.imweb.io/topic/598973c2c72aa8db35d2e291](https://www.imweb.io/topic/598973c2c72aa8db35d2e291)
