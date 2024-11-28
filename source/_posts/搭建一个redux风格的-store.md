title: 搭建一个redux风格的 store
author: Billy Yin
tags: []
categories: []
date: 2018-12-10 00:16:00
---


上一篇文章 我们已经实现了 ```context```, 实现了简易的多层次组件共享状态。

* 问题： ```context``` 中的状态的更新不是受控的 / 操作不便

```javascript

class App extends Component {
  constructor() {
    super()
    this.state = {
      count: 100
    }
  }

  getChildContext() {
    return {  
        count: this.state.count
    }
  }
  render(){
      ...
  }
}

```

* 状态管理容器 

创建 store.js

```javascript

function createStore(reducer) {
    let state = null; //初始化state
    let listeners = []; //监听变化的回调函数队列
    let subscribe = (listener) => listeners.push(listener); // 定义并随后暴露添加绑定回调函数的方法
    let getState = () => state;// 定义并暴露获取读取 state 的唯一方法
    let dispatch = (action) => {
        state = reducer(state, action);//根据action改变状态
        listeners.forEach((listener) => listener())//把listeners都执行一次
    }
    dispatch({})
    return { subscribe, getState, dispatch }
}

// 改变状态的规则函数
function reducer(state, action) {
    if (!state) {
        state = {
            number: 1
        }
    }
    switch (action.type) {
        case 'AddNumber':
            state.number++;
            break;
        case 'MinusNumber':
            state.number--;
            break;
        default:
    }
    return { ...state }
}

export const store = createStore(reducer)
```

* 在最外层组件中

```js
import { store } from './store';

class App extends Component {
// ...
  getChildContext() {
    return { store }
  }

  static childContextTypes = {
    store: PropTypes.object,
  }
//...
}
```

* 任意在子组件中

```js
class ChildComponent extends Component {
    constructor() {
        super()
        this.state = {}
    }
    //...
    componentDidMount() {
        const { store } = this.context
        this._updateState()
        store.subscribe(() => this._updateState()) // 添加dispatch的回调函数，自动更新最新的数据
    }

    _updateState() {
        const { store } = this.context;
        const state = store.getState();
        this.setState(state)
    }

    render(){
        //...
    }
}
```

如此一来， 所有关于store的状态更新必然通过dispatch，做到了受控的状态更新