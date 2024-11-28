title: ES6 Proxy
author: Billy Yin
tags: []
categories: []
date: 2019-11-07 00:10:00
---
### 概念
Proxy 可以理解成，在目标对象之前架设一层“拦截”，外界对该对象的访问，都必须先通过这层拦截，因此提供了一种机制，可以对外界的访问进行过滤和改写。

### 语法
let p = new Proxy(target, handler);

###### set
```
const obj = { name: '' }

const proxy = new Proxy(obj, {
    set: (target, prop, value) => {
        target[prop] = value.toString().toUpperCase();
    },
})

proxy.name = 'a';

console.log(obj)
console.log(proxy)
```
###### get
```
const obj = {
    name: 'a'
}

const proxy = new Proxy(obj, {
    get: (target, prop) => {
        return target[prop].toString().toUpperCase()
    }
})

console.log(proxy.name)
```
##### has
```
const obj = { name: '' }

const proxy = new Proxy(obj, {
    has: (target, prop) => {
        console.log(`call ${prop} values ${target[prop]}`)
        return target[prop] !== undefined;
    },
})

console.log('name' in proxy)
```
##### deleteProperty
```
const obj = {
    a: 1,
    b: 2
}

const proxy = new Proxy(obj, {
    deleteProperty: (target, prop) => {
        if (prop === 'b') {
            return false
        }
        delete target[prop]
        return true
    }
})

delete proxy.b

console.log(obj)

delete proxy.a

console.log(obj)
```

##### apply
```
function sum(a, b) {
    return a + b;
}
const absSum = new Proxy(sum, {
    apply(target, thisArg, args) {
        const value = target.apply(thisArg, args);
        return value < 0 ? -value : value;
    }
});

console.log(absSum(-1, -2));
```
##### 更多API见[https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy#](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy#)


### Proxy 相比于 Object.defineProperty 优势
##### 监听数组事件
```
const obj = {}
var list = [1, 2, 3]
Object.defineProperty(obj, 'list', {
    get: () => {
        return list
    },
    set: (newVal) => {
        console.log(newVal)
        list = newVal
    }
})
obj.list.push(4)
obj.list = [1, 2, 3, 4, 5]
```
```
const list = [1, 2, 3]

var proxyObj = new Proxy(list, {
    set: (target, property, value) => {
        console.log('set')
        console.log(property, value)
        return Reflect.set(target, property, value);
    }
})

proxyObj.push(4)
```
Reflect对象与Proxy对象一样，也是ES6为了操作对象而提供的新API。
 详见: [https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Reflect](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Reflect)
##### 监听的是一整个对象而非对象的某个属性

>let p = new Proxy(target, handler);

vs

>Object.defineProperty(obj, prop, descriptor)

##### 支持嵌套代理
```
let obj = {
    info: {
        name: 'eason',
        blogs: ['webpack', 'babel', 'cache']
    }
}
let handler = {
    get(target, key) {
        console.log('get', key)
        // 递归创建并返回
        if (typeof target[key] === 'object' && target[key] !== null) {
            return new Proxy(target[key], handler)
        }
        return Reflect.get(target, key)
    },
    set(target, key, value) {
        console.log('set', key, value)
        return Reflect.set(target, key, value)
    }
}
let proxy = new Proxy(obj, handler)
// 以下两句都能够进入 set
proxy.info.name = 'Zoe'
proxy.info.blogs.push('proxy')
```
### 使用场景
##### 实现观察者模式

> Vue 3.0 使用 Proxy 代替 Object.defineProperty 实现双向绑定

##### 数据校验
```
const obj = {
    phone: '13888888888'
}

const proxy = new Proxy(obj, {
    set: (target, prop, value) => {
        if (prop === 'phone' && !/^(\+86)?1[\d]{10}$/.test(value)) {
            return false;
        }
        target[prop] = value;
        return true;
    }
})

proxy.phone = 'a'

console.log(proxy)

proxy.phone = '13899999999'

console.log(proxy)
```

##### 添加实用方法
```
const list = [
    { name: 'a', phone: '13888888888' },
    { name: 'b', phone: '13899999999' },
];

const proxyList = new Proxy(list, {
    get: (target, prop) => {
        if (prop in target) {
            return target[prop];
        }

        for (const item of target) {
            if (item.name === prop) {
                return item;
            }
        }
        return undefined;
    }
});

console.log(proxyList[0]);
console.log(proxyList["a"]);
```
 ### Proxy的坑

##### this指向问题
```
const target = {
    m: function () {
        console.log(this === target);
    }
};
const proxy = new Proxy(target, {});
target.m() // true
proxy.m() // false
```
解决
```
const target = {
    m: function () {
        console.log(this === target);
    }
};
const proxy = new Proxy(target, {
    get(target, prop) {
        return target[prop].bind(target);
    }
});
target.m() // true
proxy.m() // true
```




