title: React High Order Component
author: Billy Yin
tags: []
categories: []
date: 2019-07-25 22:45:00
---
### 高阶函数：接受一个或多个函数并返回一个函数
```
var add = function (x) {
      return function (y) {
          return x + y;
      }
 }
add(1)(2); // 3
```
JS中，array的map、filter、reduce等方法就是高阶函数的例子
```
var numbers = [1, 2, 3, 4];
function double(num) {
    return num * 2;
}
var doubledNumbers = numbers.map(double); //[2,4,6,8]
```

#### 高阶函数的应用---柯里化
一道经典面试题
>写一个add方法实现以下效果
```
add(1) //1
add(1)(2) //3
add(1)(2)(3) //6
add(1)(2)(3)(4) //10
```

> 答案之一:

```
function add(a) {
    function sum(b) { // 使用闭包
    	a = a + b; // 累加
    	return sum;
    }
    sum.toString = function() { // 重写toString()方法
        return a;
    }
    return sum; // 返回一个函数
}

```
这里的add就是高阶函数

### 高阶组件：接受一个组件并返回一个组件

实现方式 1 属性代理

```
	const Container = (WrappedComponent) =>
		class extends React.Component {
			render() {
				let newProps = { status: 'ok' }
				return <WrappedComponent {...this.props} {...newProps} />
			}
		}
```
实现方式 2 反向继承
```
	const Container = (WrappedComponent) =>
		class extends WrappedComponent {
			render() {
				return super.render()
			}
		}
```

> Tips: super语法：

1  super([arguments]); // 访问父对象上的构造函数

2  super.functionOnParent([arguments]); // 访问对象上的方法


高阶组件使用方式 

```
class App extends React.Component {
  render() {
    return (
      <div>
        ...
      </div>
    )
  }
}

export default Container(App);
```
or Decorator方式
```
@Container
class App extends React.Component {
  render() {
    return (
      <div>
        ...
      </div>
    )
  }
}
```
>Tips: ES6装饰器的说明：

```
@decorator
class A {}

// 等同于

class A {}
A = decorator(A) || A;
```

高阶组件的应用：
* 控制props
* 渲染劫持
* 控制state
* 为组件增加统一的样式或布局
* 。。。

### 高阶组件应用之react-redux里的connect
```
import React, { Component } from 'react'
import PropTypes from 'prop-types'

const connect = (mapStateToProps)=>(WrappedComponent) => {
  class Connect extends Component {
    static contextTypes = {
      store: PropTypes.object
    }

    constructor () {
      super()
      this.state = { allProps: {} }
    }

    componentWillMount () {
      const { store } = this.context
      this._updateProps()
      store.subscribe(() => this._updateProps())
    }

    _updateProps () {
      const { store } = this.context
      let stateProps = mapStateToProps(store.getState())
      stateProps.dispatch=store.dispatch
      this.setState({
        allProps: { // 整合普通的 props 和从 state 生成的 props
          ...stateProps,
          ...this.props
        }
      })
    }
    render () {
      return <WrappedComponent {...this.state.allProps}/>
    }
  }

  return Connect
}
export default connect
```


使用connect

```
import React, { PureComponent } from 'react';
import { connect } from 'dva';
import LoginForm from './LoginForm';

@connect(({ login, loading }) => ({
    login,
    loading,
}))
export default class Login extends PureComponent {
    render() {
        const { dispatch, loading } = this.props;
        return (
            <LoginForm
                isLoading={loading.effects['login/login']}
                onOk={(data) => {
                    dispatch({
                        type: 'login/login',
                        payload: {
                            ...data,
                        },
                    });
                }}
            />
        );
    }
```

注意：
1、不要在render中构建高阶组件

```
render() {
  // 每次调用 render 函数都会创建一个新的 EnhancedComponent
  // EnhancedComponent1 !== EnhancedComponent2
  const EnhancedComponent = enhance(MyComponent);
  // 这将导致子树每次渲染都会进行卸载，和重新挂载的操作！
  return <EnhancedComponent />;
}
```

2、注意添加原组件的静态方法

```
function enhance(WrappedComponent) {
  class Enhance extends React.Component {/*...*/}
  // 必须准确知道应该拷贝哪些方法 :(
  Enhance.staticMethod = WrappedComponent.staticMethod;
  return Enhance;
}
```
3、ref和key不会被传递