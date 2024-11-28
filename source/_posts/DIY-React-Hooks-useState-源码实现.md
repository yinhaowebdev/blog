title: DIY React Hooks + useState 源码实现
author: Billy Yin
tags: []
categories: []
date: 2020-09-02 15:26:00
---
我作为一个函数组件，怎么就实现内部状态和类生命周期了呢？
现在学习一番
### useState
```
import React from 'react';
import ReactDOM from 'react-dom';

let currentState

const render = () => {
    ReactDOM.render(
        <App/>,
      document.getElementById('root')
    );
}

function useState(initialState){
    let state = currentState || initialState 
    const setState=(targetState)=>{
        currentState = targetState;
        render()
    }
    return [state, setState]
}

function App(){
    const [count, setCount] = useState(1)
    const add = ()=>{
		setCount(count+1)
	}
    return <div>
        <div>{count}</div>
        <div onClick={add}>add</div>
    </div>
}

ReactDOM.render(
    <App/>,
  document.getElementById('root')
);
```
### useEffects
```
// 修改上一个例子
let depsArray

function useEffect(cb,deps){
    const noDeps = !deps
    const noChanges = deps && depsArray ? !deps.every((e,index)=> e === depsArray[index]) : true;
    if(noDeps || noChanges){
        cb()
        depsArray = deps
    }
}

function App(){
    const [count, setCount] = useState(1)
    useEffect(()=>{
        console.log(123)
    },[count])
    const add = ()=>{
		setCount(count+1)
	}
    return <div>
        <div>{count}</div>
        <div onClick={add}>add</div>
    </div>
}

```
### 多个hooks的情况
```
import React from 'react';
import ReactDOM from 'react-dom';

let memoizedState = [];

let cursor = 0; 

const render = () => {
    cursor = 0
    ReactDOM.render(
        <App/>,
      document.getElementById('root')
    );
}

function useState(initialState){
    memoizedState[cursor] = memoizedState[cursor] === undefined ? initialState : memoizedState[cursor] 
    const currentCursor = cursor;
    const setState=(targetState)=>{
        memoizedState[currentCursor] = targetState;
        render()
    }
    
    return [memoizedState[cursor++], setState]
}


function useEffect(cb,deps){
    const noDeps = !deps
    const hasChanges = deps && memoizedState[cursor] ? !deps.every((e,index)=> e === memoizedState[cursor][index]) : true;
    if(noDeps || hasChanges){
        cb()
        memoizedState[cursor] = deps
    }
    cursor++
}

function App(){
    const [count, setCount] = useState(1)
    useEffect(()=>{
        console.log(123)
    },[count])
    const add = ()=>{
		setCount(count+1)
	}
    return <div>
        <div>{count}</div>
        <div onClick={add}>add</div>
    </div>
}

ReactDOM.render(
    <App/>,
  document.getElementById('root')
);
```
### React实际的实现
首次渲染和再次渲染时，逻辑是不一样的
```
// react-reconciler/src/ReactFiberHooks.js
// Mount 阶段Hooks的定义
const HooksDispatcherOnMount: Dispatcher = {
  useEffect: mountEffect,
  useReducer: mountReducer,
  useState: mountState,
 // 其他Hooks
};

// Update阶段Hooks的定义
const HooksDispatcherOnUpdate: Dispatcher = {
  useEffect: updateEffect,
  useReducer: updateReducer,
  useState: updateState,
  // 其他Hooks
};
```
##### 创建时
```
// react-reconciler/src/ReactFiberHooks.js
function mountState (initialState) {
  // 获取当前的Hook节点，同时将当前Hook添加到Hook链表中
  const hook = mountWorkInProgressHook();
  if (typeof initialState === 'function') {
    initialState = initialState();
  }
  hook.memoizedState = hook.baseState = initialState;
  // 声明一个链表来存放更新
  const queue = (hook.queue = {
    last: null,
    dispatch: null,
    lastRenderedReducer,
    lastRenderedState,
  });
  // 返回一个dispatch方法用来修改状态，并将此次更新添加update链表中
  const dispatch = (queue.dispatch = (dispatchAction.bind(
    null,
    currentlyRenderingFiber,
    queue,
  )));
  // 返回当前状态和修改状态的方法 
  return [hook.memoizedState, dispatch];
}
```
Hook和Hook链表的结构
```
function mountWorkInProgressHook(): Hook {
  const hook: Hook = {
    memoizedState: null,
    baseState: null,
    queue: null,
    baseUpdate: null,
    next: null,
  };
  if (workInProgressHook === null) {
    // 当前workInProgressHook链表为空的话，
    // 将当前Hook作为第一个Hook
    firstWorkInProgressHook = workInProgressHook = hook;
  } else {
    // 否则将当前Hook添加到Hook链表的末尾
    workInProgressHook = workInProgressHook.next = hook;
  }
  return workInProgressHook;
}
```
每次dispatch，创建一个update对象保存本次更新的信息，并且添加到hook对应的quene：
```
// react-reconciler/src/ReactFiberHooks.js
// 去除特殊情况和与fiber相关的逻辑
function dispatchAction(fiber,queue,action,) {
    const update = {
      action,
      next: null,
    };
    // 将update对象添加到循环链表中
    const last = queue.last;
    if (last === null) {
      // 链表为空，将当前更新作为第一个，并保持循环
      update.next = update;
    } else {
      const first = last.next;
      if (first !== null) {
        // 在最新的update对象后面插入新的update对象
        update.next = first;
      }
      last.next = update;
    }
    // 将表头保持在最新的update对象上
    queue.last = update;
   // 进行调度工作
    scheduleWork();
}
```

##### 更新时
```
// react-reconciler/src/ReactFiberHooks.js
function updateState(initialState) {
  return updateReducer(basicStateReducer, initialState);
}
```
basicStateReducer 的实现，兼容了useReducer
```
// react-reconciler/src/ReactFiberHooks.js
function basicStateReducer(state, action){
  return typeof action === 'function' ? action(state) : action;
} 
```
```
// react-reconciler/src/ReactFiberHooks.js
// 去掉与fiber有关的逻辑

function updateReducer(reducer,initialArg,init) {
  const hook = updateWorkInProgressHook();
  const queue = hook.queue;

  // 拿到更新列表的表头
  const last = queue.last;

  // 获取最早的那个update对象
  first = last !== null ? last.next : null;

  if (first !== null) {
    let newState;
    let update = first;
    do {
      // 执行每一次更新，去更新状态
      const action = update.action;
      newState = reducer(newState, action);
      update = update.next;
    } while (update !== null && update !== first);

    hook.memoizedState = newState;
  }
  const dispatch = queue.dispatch;
  // 返回最新的状态和修改状态的方法
  return [hook.memoizedState, dispatch];
}
```