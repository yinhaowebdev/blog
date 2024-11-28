title: redux-thunk
author: Billy Yin
tags: []
categories: []
date: 2019-07-30 11:50:58
---
### Redux数据流
view -> dispatch-> action -> reducer -> state

### 中间件(MiddleWare)

>它提供的是位于 action 被发起之后，到达 reducer 之前的扩展点。 你可以利用 Redux middleware 来进行日志记录、创建崩溃报告、调用异步接口或者路由等等。

view -> ||```被扩展的部分``` dispatch-> action -> ```被扩展的部分```||reducer -> state

一个简单的中间件
```js
const logger = store => next => action => {
    console.log('dispatch',action)
    next(action)
    console.log('finish' , action)
}
```

中间件的使用

```js
createStore(store,null, applyMiddleware(logger))
```
返回一个增强后的createStore

createStore源码
```js
export default function createStore(reducer, preloadedState, enhancer) {
  // 如果 preloadedState类型是function，enhancer类型是undefined，那认为用
  // 户没有传入preloadedState，就将preloadedState的值传给  
  // enhancer，preloadedState值设置为undefined
  if (typeof preloadedState === 'function' && typeof enhancer === 'undefined') {
    enhancer = preloadedState
    preloadedState = undefined
  }
  // enhancer类型必须是一个function
  if (typeof enhancer !== 'undefined') {
    if (typeof enhancer !== 'function') {
      throw new Error('Expected the enhancer to be a function.')
    }
    // 返回使用enhancer增强后的store
    return enhancer(createStore)(reducer, preloadedState)
  }
  //......
}
```

applyMiddleware源码

```js
export default function applyMiddleware(...middlewares) {
  return (createStore) => (reducer, preloadedState, enhancer) => {
    const store = createStore(reducer, preloadedState, enhancer)
    let dispatch = store.dispatch
    let chain = []  

    const middlewareAPI = {
      getState: store.getState,
      dispatch: (...args) => dispatch(...args)
    }
    chain = middlewares.map(middleware => middleware(middlewareAPI))
    dispatch = compose(...chain)(store.dispatch) // 对dispatch方法封装

    return {
      ...store,
      dispatch
    }
  }
}
```


### redux-thunk

>Redux Thunk middleware allows you to write action creators that return a function instead of an action. The thunk can be used to delay the dispatch of an action, or to dispatch only if a certain condition is met. The inner function receives the store methods dispatch and getState as parameters.

使用

>cnpm install redux-thunk

```js
import thunk from 'redux-thunk'
//...
createStore(store,null applyMiddleware(thunk))
```

示例
```js
const INCREMENT_COUNTER = 'INCREMENT_COUNTER';
 
function increment() {
  return {
    type: INCREMENT_COUNTER
  };
}
 
function incrementAsync() {
  return dispatch => {
    setTimeout(() => {
      // Yay! Can invoke sync or async actions with `dispatch`
      dispatch(increment());
    }, 1000);
  };
}
```


源码
```js
function createThunkMiddleware(extraArgument) {
  return ({ dispatch, getState }) => next => action => {
    if (typeof action === 'function') {
      return action(dispatch, getState, extraArgument);
    }

    return next(action);
  };
}

const thunk = createThunkMiddleware();
thunk.withExtraArgument = createThunkMiddleware;

export default thunk;

```
