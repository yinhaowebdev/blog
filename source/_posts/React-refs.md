title: React - refs
author: Billy Yin
tags: []
categories: []
date: 2020-08-14 15:58:00
---
> Refs 提供了一种方式，允许我们访问 DOM 节点或在 render 方法中创建的 React 元素。

### 方式一： createRef （>= React16.3）
一般在构造函数中将refs分配给实例属性，以供组件的其它方法中使用

* 对于HTML elements：可以获取元素实例
```
class App extends React.Component{
    constructor(props) {
        super(props);
        this.myRef = React.createRef();
        console.log(this.myRef)
      }
      componentDidMount(){
        this.myRef.current.innerHTML = '1'
      }
      render() {
        return <div ref={this.myRef} />;
      }
}
```
* 对于Class Components：可以获取组件类的实例
```
class App extends React.Component{
    constructor(props) {
        super(props);
        this.myRef = React.createRef();
        console.log(this.myRef)
      }
      componentDidMount(){
        this.myRef.current.someMethod() // 也就是说，可以允许父组件调用子组件的方法
      }
      render() {
        return <Children ref={this.myRef} />;
      }
}
class Children extends React.Component{
    someMethod(){
        console.log(123)
    }
      render() {
        return <span>123</span>;
      }
}
```
* 对于Function Components：无法获取

![function components case](https://upload-images.jianshu.io/upload_images/9594241-bdcc19fca0704681.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### 方式二： 回调Refs

React 将在组件挂载时，会调用 ref 回调函数并传入 DOM 元素，当卸载时调用它并传入 null。在 componentDidMount 或 componentDidUpdate 触发前，React 会保证 refs 一定是最新的。

```
class App extends React.Component{
    constructor(props) {
        super(props);
        this.targetRef = null
        this.myRef = (e)=> this.targetRef = e;
      }
      componentDidMount(){
        if(this.targetRef){
            this.targetRef.innerHTML = '123'
        }
      }
      render() {
        return <div ref={this.myRef} />;
      }
}
```

### 方式三：String 类型的 Refs (不推荐使用)
>它已过时并可能会在未来的版本被移除。
```
class StringRef extends Component {
  componentDidMount() {
    console.log('this.refs', this.refs);
  }
  render() {
    return (
      <div ref="container">
        StringRef
      </div>
    )
  }
}
```

### 方式四: useRef (React Hooks)
```
import { useRef } from 'react';
function RefExample(props){
    const inputElement = useRef()
    return(<div>
        <input ref={inputElement}></input>
        <button onClick={()=>{
            inputElement.current.focus()
        }}>focus</button>
    </div>)
}
```

##### useRef vs createRef
```
function RefExample(props){
    const counterUseRef = useRef()
    const counterCreateRef = createRef()
    const [counter, setcounter] = useState(0)
    if(!counterUseRef.current){
        counterUseRef.current = counter
    }
    if(!counterCreateRef.current){
        counterCreateRef.current = counter
    }
    return(<div>
        <div>{counter}</div>
        <div>{counterUseRef.current}</div>
        <div>{counterCreateRef.current}</div>
        <div onClick={()=>{
            setcounter(counter + 1)
        }}>add</div>
        </div>)
}

```
createRef 每次渲染都会返回一个新的引用，而 useRef 每次都会返回相同的引用。

### forwardRef
```
const FancyButton = React.forwardRef((props, ref) => (
  <button ref={ref} className="FancyButton">
    {props.children}
  </button>
));

// You can now get a ref directly to the DOM button:
const ref = React.createRef();
<FancyButton ref={ref}>Click me!</FancyButton>;
```
### 使用场景举例
* 管理焦点，文本选择或媒体播放。
* 触发强制动画。
* 集成第三方 DOM 库。
总而言之，都是需要获取到DOM实例的场景

### 谨慎使用
能使用react数据流完成的操作，尽量不依赖ref
因为这样的方式，很不 “react”