title: useRef & useMemo & useCallback
author: Billy Yin
tags: []
categories: []
date: 2020-08-30 17:15:00
---
### useRef 
基本用法
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
与 `createRef` 不同， 它不会每次渲染都新生成一个新的引用，而是会一直指向相同的引用

trick1：用引用的方式让回调函数获得最新的状态
```
function RefExample(props) {
    const [counter, setcounter] = useState(0)

    function handleClick() {
        setTimeout(() => {
            console.log(counter)
        }, 3000)
    }
    return (<div>
        <div>{counter}</div>
        <div onClick={handleClick}>delay show</div>
        <div onClick={() => {
            setcounter(counter + 1)
        }}>add</div>
    </div>)
}
```
```
function RefExample(props) {
    const counterUseRef = useRef()
    const [counter, setcounter] = useState(0)

    useEffect(() => {
        counterUseRef.current = counter
    })
    function handleClick() {
        setTimeout(() => {
            console.log(counterUseRef.current)
        }, 3000)
    }
    return (<div>
        <div>{counter}</div>
        <div onClick={handleClick}>delay show</div>
        <div onClick={() => {
            setcounter(counter + 1)
        }}>add</div>
    </div>)
}
```
trick2 ：保存上次状态
```
function RefExample(props) {
    const counterUseRef = useRef()
    const [counter, setcounter] = useState(0)
    useEffect(() => {
        counterUseRef.current = counter
    })

    return (<div>
        <div>{counter}</div>
        <div>{counterUseRef.current}</div>
        <div onClick={() => {
            setcounter(counter + 1)
        }}>add</div>
    </div>)
}
```
### useMemo
React 中没有 computed 这种属性，所以类似的计算属性需要手动维护
Class中修改state导致render中的计算逻辑：
```
class MemoExample extends React.Component {
    constructor() {
        super()
        this.state = {
            counter: 0,
            str: '1'
        }
    }
    render() {
        const doubleCounter = this.state.counter * 2
        return (<div>
            <div>{this.state.counter}</div>
            <div>{doubleCounter}</div>
            <div onClick={() => {
                this.setState({ counter: this.state.counter + 1 })
            }}>add</div>
            <div>{this.state.str}</div>
            <div onClick={() => {
                this.setState({ str: this.state.str + '1' })
            }}>add str</div>
        </div>)
    }
}
```
function中可以使用useMemo来避免不必要的计算：
```
function MemoExample() {
    const [counter, setcounter] = useState(0)
    const [str, setStr] = useState('1')
    const doubleCounter = useMemo(() => {
        console.log('useMemo')
        return counter * 2
    }, [counter])
    return (<div>
        <div>{counter}</div>
        <div>{doubleCounter}</div>
        <div onClick={() => {
            setcounter(counter + 1)
        }}>add</div>
        <div>{str}</div>
        <div onClick={() => {
            setStr(str + '1')
        }}>add str</div>
    </div>)
}
```

### useCallBack
一个容易被忽略的细节：将函数直接写进子组件的props中会导致每次render都生成新的函数，导致子组件用shouldComponentUpdate或React.memo进行props检测的时候每次都会检测到新的参数变化，无法阻止事实上不必要的重新渲染
```
class App extends React.Component {
    constructor() {
        super()
        this.state = {
            count: 1,
            name: 'jack'
        }
    }
    componentDidMount() {
        setInterval(() => this.setState(function (state, props) {
            return {
                count: state.count + 1
            };
        }), 1000)
    }

    render() {
        return (
            <div>
                <Child someProps={() => { console.log(123) }} name={this.state.name} />
            </div>
        );
    }
}

class Child extends React.PureComponent {
    render() {
        console.log('render') // 每秒钟渲染一次
        return (<div>
            {this.props.name}
        </div>)
    }
}
```
流行的一个优化技巧是：
```
class App extends React.Component {
    constructor() {
        super()
        this.state = {
            count: 1,
            name: 'jack'
        }
    }
    componentDidMount() {
        setInterval(() => this.setState(function (state, props) {
            return {
                count: state.count + 1
            };
        }), 1000)
    }

    somePropsCallback() { // 不要使用inline的方式书写props中的回调函数
        console.log(123)
    }

    render() {
        return (
            <div>
                <Child someProps={this.somePropsCallback} name={this.state.name} />
            </div>
        );
    }
}
```
useCallback可以将函数缓存下来，只要依赖参数不发生变化，那么指向的都是同一个function:
```
function App() {
    const [count, setCount] = useState(1)
    const [name, setName] = useState('jack')
    useEffect(() => {
        setTimeout(() => { setCount(count + 1) }, 1000)
    }, [count])
    const somePropsCallback = useCallback(() => {
        console.log(123)
    }, [])
    return (
        <div>
            {count}
            <Child someProps={somePropsCallback} name={name} />
        </div>
    )
}
class Child extends React.PureComponent {
    render() {
        console.log('render')
        return (<div>
            {this.props.name}
        </div>)
    }
}

```

### 一个问题
##### 困境
```
function Form() {
  const [text, updateText] = useState('');

  const handleSubmit = useCallback(() => {
    console.log(text);
  }, [text]);

  return (
    <>
      <input value={text} onChange={(e) => updateText(e.target.value)} />
      <ExpensiveTree onSubmit={handleSubmit} /> // 很重的组件
    </>
  );
}
```
##### 解决
```
function Form() {
  const [text, updateText] = useState('');
  const textRef = useRef();

  useEffect(() => {
    textRef.current = text; // Write it to the ref
  });

  const handleSubmit = useCallback(() => {
    const currentText = textRef.current; // Read it from the ref
    alert(currentText);
  }, [textRef]); // Don't recreate handleSubmit like [text] would do

  return (
    <>
      <input value={text} onChange={e => updateText(e.target.value)} />
      <ExpensiveTree onSubmit={handleSubmit} />
    </>
  );
}
```
##### 封装成自定义hook
```
function Form() {
  const [text, updateText] = useState('');
  // Will be memoized even if `text` changes:
  const handleSubmit = useEventCallback(() => {
    alert(text);
  }, [text]);

  return (
    <>
      <input value={text} onChange={e => updateText(e.target.value)} />
      <ExpensiveTree onSubmit={handleSubmit} />
    </>
  );
}

function useEventCallback(fn, dependencies) {
  const ref = useRef(() => {
    throw new Error('Cannot call an event handler while rendering.');
  });

  useEffect(() => {
    ref.current = fn;
  }, [fn, ...dependencies]);

  return useCallback(() => {
    const fn = ref.current;
    return fn();
  }, [ref]);
}
```