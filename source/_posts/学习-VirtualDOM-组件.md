title: 学习 VirtualDOM - 组件
author: Billy Yin
tags: []
categories: []
date: 2019-06-28 13:34:00
---
###  组件有两种写法

```javascript
// 写法 1：继承React.Component
class A extends React.Component {
  constructor(props){
    super(props)
    this.state={
      count : 1
    }
  }
  render() {
    return(<div>I'm componentA, my count is { this.state.count }</div>)
  }
}

// 写法 2：无状态组件
const A = () => <div>I'm componentA</div>
```

createElement() 的参数发生了变化


### 真正的渲染之前，包装一层预处理

```
function render(vdom, container, index = 0) {
  let component
  if (_.isFunction(vdom.nodeName)) { // 如果是组件
    if (vdom.nodeName.prototype.render) { // Class组件
      component = new vdom.nodeName(vdom.attributes)
    } else {
      component = vdom.nodeName(vdom.attributes) // 处理无状态组件
    }
  }
  component ? _render(component, container, index) : _render(vdom, container, index)
}
```

### _render
```
function _render(component, container, index) {
  const vdom = component.render ? component.render() : component 
  if (_.isString(vdom) || _.isNumber(vdom)) { 
    container.innerText = container.innerText + vdom 
    return
  }
  const dom = document.createElement(vdom.nodeName)
  for (let attr in vdom.attributes) {
    setAttribute(dom, attr, vdom.attributes[attr])
  }
  vdom.children.forEach((vdomChild, index) => render(vdomChild, dom, index))
  if (component.container) // 调用 setState 方法时
  { 
    component.container.childNodes[component.index].innerHTML = null
    component.container.childNodes[component.index].appendChild(dom)
    return
  }
  component.container = container
  component.index = index

  container.appendChild(dom)
}

```
### state and props
组件的其他实例化特性由_render提供

```
function Component(props) {
  this.props = props
  this.state = this.state || {}
}
```
### setState

setstate 直接调用_render 把component实例传进去
```
Component.prototype.setState = function (updateObj) {
  this.state = Object.assign({}, this.state, updateObj)
  _render(this) // 重新渲染
}
```