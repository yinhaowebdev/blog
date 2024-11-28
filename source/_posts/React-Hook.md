title: React Hook
author: Billy Yin
tags: []
categories: []
date: 2019-07-12 11:56:00
---
### React 声明组件的方式

Class声明和函数式声明(无状态组件)。
Class声明可以操作state、生命周期等
函数式声明不可以

Hook 是 React 16.8 的新增特性。它可以让你在不编写 class 的情况下使用 state 以及其他的 React 特性。
### 原则
· 只在最顶层调用Hook

· 只在React函数中调用Hook

### state Hook
如下组件
```js
class Example extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    };
  }

  render() {
    return (
      <div>
        <p>You clicked {this.state.count} times</p>
        <button onClick={() => this.setState({ count: this.state.count + 1 })}>
          Click me
        </button>
      </div>
    );
  }
}
```
可以写成
```js
import React, { useState } from 'react';

function Example() {
  // 声明一个叫 "count" 的 state 变量
  const [count, setCount] = useState(0);

  return (
    <div>
      // 获取 state
      <p>You clicked {count} times</p>
      // 改变 state
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```
### effect Hook
如下代码
```
class Example extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    };
  }

  componentDidMount() {
    document.title = `You clicked ${this.state.count} times`;
  }

  componentDidUpdate() {
    document.title = `You clicked ${this.state.count} times`;
  }

  render() {
    return (
      <div>
        <p>You clicked {this.state.count} times</p>
        <button onClick={() => this.setState({ count: this.state.count + 1 })}>
          Click me
        </button>
      </div>
    );
  }
}
```
可以写成
```
import React, { useState, useEffect } from 'react';

function Example() {
  const [count, setCount] = useState(0);

  // Similar to componentDidMount and componentDidUpdate:
  useEffect(() => {
    // Update the document title using the browser API
    document.title = `You clicked ${count} times`;
  });

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

### 生命周期

```
import React, { useState, useEffect } from 'react';

class App extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    };
  }

  render() {
    return (
      <div>
        <p>You clicked {this.state.count} times</p>
        <button onClick={() => this.setState({ count: this.state.count + 1 })}>
          Click me!!!!
        </button>
        {
          this.state.count < 4 ? <Example></Example> : null
        }

      </div>
    );
  }
}

function Example() {
  const [count, setCount] = useState(0);

  // Similar to componentDidMount and componentDidUpdate:
  useEffect(() => {
    // Update the document title using the browser API
    document.title = `You clicked ${count} times`;
    return function () {
      document.title = `React App`
    };
  });

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
export default App;

```
组件的每一次更新都要调用Effects

创建时：调用Effects中的回调

更新时：先调用Effects中回调的return的回调，再调用Effects中的回调

卸载时：调用Effects中回调的return的回调

### 自定义Hook ， 逻辑的抽离

```
import React, { useState, useEffect } from 'react';



function useCount(friendID) {
  const [count, setCount] = useState(0);

  // Similar to componentDidMount and componentDidUpdate:
  useEffect(() => {
    console.log(1)
    // Update the document title using the browser API
    document.title = `You clicked ${count} times`;
  });
  return { setCount, count };
}
class App extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    };
  }

  render() {
    return (
      <div>
        <p>You clicked {this.state.count} times</p>
        <button onClick={() => this.setState({ count: this.state.count + 1 })}>
          Click me!!!!
        </button>
        {
          this.state.count < 4 ? <Example></Example> : null
        }
        {
          this.state.count < 4 ? <Example1></Example1> : null
        }

      </div>
    );
  }
}

function Example() {
  const { count, setCount } = useCount()

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}

function Example1() {
  const { count, setCount } = useCount()

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
export default App;

```

· 自定义 Hook 必须以 “use” 开头
· 自定义 Hook被不同的组件使用时state和effects是独立的