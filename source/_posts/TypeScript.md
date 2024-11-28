title: TypeScript
author: Billy Yin
tags: []
categories: []
date: 2019-07-19 15:52:00
---
### TS 介绍
TypeScript 是 JavaScript 的一个超集，支持 ECMAScript 6 标准。
TypeScript 设计目标是开发大型应用，它可以编译成纯 JavaScript，编译出来的 JavaScript 可以运行在任何浏览器上。

### TS编译运行
将 TS 编译成 JS后运行
1、 使用tsc
>cnpm install -g typescript

>tsc hello.ts

2、ts-loader
```
// webpack.config.prod
//...
            test: /\.(ts|tsx)$/,
            use: [
              {
                loader: require.resolve('ts-loader'),
//...
```
### TS 基本语法
TS支持所有的JS语法
```
const hello : string = "Hello World!"
```
#### 基本类型

boolean
number
string
array
enum
```

enum Color {Red, Green, Blue}
let c: Color = Color.Green;

console.log(c); // 获取值，相当于 console.log(Color["Green"]);

enum Color1 {Red = 1, Green = 2, Blue = 4}
let c1: Color1 = Color1.Green;
console.log(c1); // 获取值，2
```
any
void
```
function warnUser(): void {
    console.log("This is my warning message");
}

warnUser();
```

#### 函数
```
function add(x:number,y:number):number{
  return x+y
}
```

```
var add:(number1:number,number2:number)=>number=function(x:number,y:number):number{
  return x+y
}
```

#### 接口
TypeScript的核心原则之一是对值所具有的结构进行类型检查。 它有时被称做“鸭式辨型法”或“结构性子类型化”。 在TypeScript里，接口的作用就是为这些类型命名和为你的代码或第三方代码定义契约。
接口不能转换为 JavaScript。 它只是 TypeScript 的一部分。
```
interface IPerson { 
    firstName:string, 
    lastName:string, 
    sayHi: ()=>string 
} 
 
var customer:IPerson = { 
    firstName:"Tom",
    lastName:"Hanks", 
    sayHi: ():string =>{return "Hi there"} 
} 
```
接口可以继承
```
interface Person { 
   age:number 
} 
 
interface Musician extends Person { 
   instrument:string 
} 
 
var drummer = <Musician>{}; 
drummer.age = 27 
drummer.instrument = "Drums" 
console.log("年龄:  "+drummer.age)
console.log("喜欢的乐器:  "+drummer.instrument)
```
#### 泛型
```
function identity<T>(arg: T): T {
    return arg;
}

let output1 = identity<string>("myString");  // type of output will be 'string'

let output2 = identity("myString");  // type of output will be 'string'
```

#### 修饰符

public（默认）
private
protected

有了修饰符就可以实现封装

### React 中使用TS
以create-react-app为例:
>cnpm install -g create-react-app
>create-react-app apptest --scripts-version=react-scripts-ts
>cd apptest
> npm run eject

```
// App.tsx
import * as React from 'react';
import C from './C'

class App extends React.Component {
  public render() {
    const params = {
      age: 1,
      name: 'abc',
    }
    return (
      <div>
        <C {...params} />
      </div>
    );
  }
}
export default App;
```

```
//C.tsx
import * as React from 'react';

interface IProps {
    name: string,
    age: number
}

interface IState {
    name: string,
    age: number
}

class C extends React.Component<IProps, IState> {
    public render() {
        return (
            <div className="App">
                name:{this.props.name}
                age:{this.props.age}
            </div>
        );
    }
}

export default C;
```

### TS 优点

提升开发体验

编译时检查类型，减少线上bug的可能性

增强前端程序员对面向对象的理解，提升代码质量

。。。