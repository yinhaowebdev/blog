title: redux-saga
author: Billy Yin
tags: []
categories: []
date: 2019-08-08 00:19:00
---
### 概念

redux-saga 是一个用于管理应用程序 Side Effect（副作用，例如异步获取数据，访问浏览器缓存等）的 library，它的目标是让副作用管理更容易，执行更高效，测试更简单，在处理故障时更容易。

### 特点

利用同步方式处理异步逻辑
api丰富，灵活操纵异步行为

### 使用方法

>cnpm install redux-saga

store
```js
import {createStore,applyMiddleware}  from "redux";
import reducer from "./reducer";
import createSagaMiddleware from "redux-saga";
import {rootSaga} from "../saga"
//执行它可以得到中间件函数
let sagaMiddleware = createSagaMiddleware();

let store = createStore(reducer,applyMiddleware(sagaMiddleware));
//开始执行rootSaga
 sagaMiddleware.run(rootSaga);
 export default store;
```
saga
```
import {takeEvery,put,all,call} from "redux-saga/effects";
const delay = ms=>new Promise((resolve,reject)=>{
   setTimeout(()=>{
       resolve()
   },ms)
});
function* add(dispatch,action) {
    yield call(delay,1000);
    yield put({type:'ADD'});
}
function* watchAdd() {
    //监听派发给仓库的动作，如果动作类型匹配的话，会执行对应的监听生成器
    yield takeEvery('ADD_ASYNC',add)
}
export function* rootSaga() {
    yield watchAdd()
}
```

### api
`put` 相当于 `dispatch`
`call` 相当于 js 里的 `call`
`take` 阻塞当前 saga，直到接收到指定的 action，代码才会继续往下执行
`fork` 类似于 `call` effect，区别在于它不会阻塞当前 saga，如同后台运行一般，它的返回值是一个 task 对象
`cancel` 针对 `fork` 方法返回的 task ，可以进行取消关闭

### 辅助函数
`takeEvery`  在发起（dispatch）到 Store 并且匹配 pattern 的每一个 action 上派生一个 saga。
`takeLatest` 在发起到 Store 并且匹配 pattern 的每一个 action 上派生一个 saga。并自动取消之前所有已经启动但仍在执行中的 saga 任务。
`takeLeading` 在发起到 Store 并且匹配 pattern 的每一个 action 上派生一个 saga。 它将在派生一次任务之后阻塞，直到派生的 saga 完成，然后又再次开始监听指定的 pattern。

### 有趣的用法

```
function* controlSaga (action) {
    const task = yield fork(getDataSaga, action); // 运行getDataSaga这个任务
    yield race([ // 利用rece谁先来用谁的原则，完美解决 超时自动取消与手动取消的 的问题
        take('CANCEL_REQUEST'), // 到这步时就阻塞住了，直到发出type为'CANCEL_REQUEST'的action，才会继续往下
        call(delay, 1000) // 控制时间
    ]);
    yield cancel(task); // 取消任务，取消后，cancelled() 会返回true，否则返回false
}
```

参考 ：

[https://juejin.im/post/5ad83a70f265da503825b2b4](https://juejin.im/post/5ad83a70f265da503825b2b4)
[https://zhuanlan.zhihu.com/p/35437092?group_id=966592505576407040](https://zhuanlan.zhihu.com/p/35437092?group_id=966592505576407040)
