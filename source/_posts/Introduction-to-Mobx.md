title: Introduction to Mobx
author: Billy Yin
tags: []
categories: []
date: 2019-08-28 00:11:00
---
### Mobx是什么

MobX实现简单、可扩展的状态管理。

使用MobX将应用变成响应式可归纳为三部曲：

>定义状态并使其可观察 (Observable)
>创建视图以响应状态的变化 (observer)
>更改状态(action)

### 安装

>cnpm install --save mobx mobx-react

### 概念

##### Observable

> Observable 值可以是JS基本数据类型、引用类型、普通对象、类实例、数组和映射。

返回相应的```Observable```对象

```js
import {observable} from 'mobx';

const map = observable.map({ key: "value"});
map.set("key", "new value");

const list = observable([1, 2, 4]);
list[2] = 3;

const person = observable({
    firstName: "Clive Staples",
    lastName: "Lewis"
});
person.firstName = "C.S.";

const temperature = observable.box(20);
temperature.set(25);
```

```js
import {observable} from 'mobx';

let appState = observable({
    timer: 0
});
```

##### observer

使用mobx-react observer() 来包裹 React 组件，形成了```observer```。


```js
import { observer } from 'mobx-react';

let App = observer(({ appState }) => {
    return (
        <div className="App">
            <h1>Time passed: {appState.timer}</h1>
        </div>
    );
});
```
or

```js
import { observer } from 'mobx-react';

@observer
class App extends React.Component {
  render() {
    return (
      <div>
         <div className="App">
            <h1>Time passed: {this.props.appState.timer}</h1>
        </div>
      </div>
    );
  }
};

ReactDOM.render(<App appState={appState} />, document.getElementById('root'));
```

##### action

```action``` 用来改变 ```observable```的状态。

```
import { observable, action } from 'mobx';
var appState = observable({
    timer: 0
});
appState.resetTimer = action(function reset() {
    appState.timer = 0
});
appState.addTimer = action(function add() {
    appState.timer++
});
let App = observer(({ appState }) => {
    return (
        <div className="App">
            <button onClick={appState.resetTimer}>
                Seconds passed: {appState.timer}
            </button>
            <button onClick={appState.addTimer}>
                +
            </button>
        </div>
    );
});
ReactDOM.render(<App appState={appState} />, document.getElementById('root'));
```

需要注意的是，observable的状态不仅仅可以通过action改变。

##### 对比Mobx和Redux

* 相比```Redux```，```Mobx``` 省略了很多代码量
* ```Redux```的状态管理更严格
* ```Mobx``` 可以设置多个store，```Redux```只能设置一个
* ```Mobx``` 对性能更友好 （```观察者模式``` vs ```Object.assign()```）

### 概念回顾

observable 类似于 store

observer 是装饰后的react组件

action 用来改变 observable 的状态


### Class声明 Observerable
```
class AppState {
    @observable timer = 2
    @action clearTimer() {
        this.timer = 0
    }
}

var appState = new AppState()
```
```
@observer
class App extends React.Component {
  componentDidMount() {
    this.props.appState.timer++

  }
  render() {
    return (
      <div>
        <div className="App">
          <h1>Time passed: {this.props.appState.timer}</h1>
        </div>
      </div>
    );
  }
};

ReactDOM.render(<App appState={appState} />, document.getElementById('root'));
```


### 组件内定义局部observerable
```
@observer
class TodoList extends React.Component {
  @observable newTodoTitle = "";

  render() {
    return (
      <div>
        <form onSubmit={this.handleFormSubmit}>
          New Todo:
          <input
            type="text"
            value={this.newTodoTitle}
            onChange={this.handleInputChange}
          />
          <button type="submit">Add</button>
        </form>
        <hr />
        <ul>
          {this.props.store.todos.map(todo => (
            <Todo todo={todo} key={todo.id} />
          ))}
        </ul>
        Tasks left: {this.props.store.unfinishedTodoCount}
      </div>
    );
  }

  @action
  handleInputChange = e => {
    this.newTodoTitle = e.target.value;
  };

  @action
  handleFormSubmit = e => {
    this.props.store.addTodo(this.newTodoTitle);
    this.newTodoTitle = "";
    e.preventDefault();
  };
}
```

### computed

```
class AppState {
    @observable timer = 2
    @computed get timessTwo() {
        return this.timer * 2
    }
    set timessTwo(timessTwo) {
        this.timer = timessTwo / 2
    }
}
```
```
@observer
class App extends React.Component {
  componentDidMount() {
    this.props.appState.timessTwo++
  }
  render() {
    return (
      <div>
        <div className="App">
          <h1>Time passed: {this.props.appState.timer}</h1>
          <h2>after times 2 :{this.props.appState.timessTwo}</h2>
        </div>
      </div>
    );
  }
};
```
### autorun & reaction
监听
```
autorun(() => {
    console.log(appState.timer)
})
```
```
const reaction1 = reaction(
    () => appState.timer ,
    timer => console.log(`reaction 1: ${timer}`)
);
```
### 开发工具
```
cnpm i mobx-react-devtools
```
```
import DevTools from "mobx-react-devtools";

{process.env.NODE_ENV !== 'production' ? <DevTools /> : null}
```

### 严格模式
```
import { configure } from "mobx";
configure({ enforceActions: true }) // 不允许在action之外进行状态修改
```

### 异步
1 runInAction
```
mobx.configure({ enforceActions: true })

class Store {
    @observable githubProjects = []
    @observable state = "pending" // "pending" / "done" / "error"

    @action
    fetchProjects() {
        this.githubProjects = []
        this.state = "pending"
        fetchGithubProjectsSomehow().then(
            projects => {
                const filteredProjects = somePreprocessing(projects)
                // 将‘“最终的”修改放入一个异步动作中
                runInAction(() => {
                    this.githubProjects = filteredProjects
                    this.state = "done"
                })
            },
            error => {
                // 过程的另一个结局:...
                runInAction(() => {
                    this.state = "error"
                })
            }
        )
    }
}
```
2 await/async
```
class Store {
    @observable githubProjects = []
    @observable state = "pending" // "pending" / "done" / "error"

    @action
    async fetchProjects() {
        this.githubProjects = []
        this.state = "pending"
        try {
            const projects = await fetchGithubProjectsSomehow()
            const filteredProjects = somePreprocessing(projects)
            // await 之后，再次修改状态需要动作:
            runInAction(() => {
                this.state = "done"
                this.githubProjects = filteredProjects
            })
        } catch (error) {
            runInAction(() => {
                this.state = "error"
            })
        }
    }
}
```
3 flow(generator function)
```
class Store {
    @observable githubProjects = []
    @observable state = "pending"

    fetchProjects = flow(function * () { // <- 注意*号，这是生成器函数！
        this.githubProjects = []
        this.state = "pending"
        try {
            const projects = yield fetchGithubProjectsSomehow() // 用 yield 代替 await
            const filteredProjects = somePreprocessing(projects)
            // 异步代码块会被自动包装成动作并修改状态
            this.state = "done"
            this.githubProjects = filteredProjects
        } catch (error) {
            this.state = "error"
        }
    })
}
```

### tip

##### 使用大量的小组件

@observer 组件会追踪它们使用的所有值，并且当它们中的任何一个改变时重新渲染。 所以你的组件越小，它们需要重新渲染产生的变化则越小;这意味着用户界面的更多部分具备彼此独立渲染的可能性。

### 脚手架
[ https://github.com/mobxjs/mobx-react-boilerplate](https://github.com/mobxjs/mobx-react-boilerplate)