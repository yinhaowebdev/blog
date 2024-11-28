title: Svelte 简介
author: Billy Yin
tags: []
categories: []
date: 2023-01-03 11:42:00
---
### 定义
> Svelte 是一种全新的构建用户界面的方法。传统框架如 React 和 Vue 在浏览器中需要做大量的工作，而 Svelte 将这些工作放到构建应用程序的编译阶段来处理。

### 性能
[跑分网站](https://krausest.github.io/js-framework-benchmark/current.html)
![图片1](https://upload-images.jianshu.io/upload_images/9594241-3aaabe85c7a76841.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![图片2](https://upload-images.jianshu.io/upload_images/9594241-26e5a9b0284cb556.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

总体来说，性能优于react和vue。

##### react/vue 为何慢？
更新时虚拟DOM diff费时，虽然react16引入了fiber，vue3引入了static hoist，但交给浏览器的工作总量大致降低不了多少。

##### sevelte如何优化

· 没有虚拟DOM
· 编译时优化

简单的说，sevelte在编译时做了很多事情。使得每一个变量得以进行脏检测，并记录变量与dom之间的关系。运行时一旦变量“脏”了，则精准计算出需要rerender哪一个dom。


### 组件
##### 父子传值
https://www.sveltejs.cn/tutorial/declaring-props
```
// index.svelte
<script>
	import Nested from './Nested.svelte';
</script>

<Nested answer={42}/>
```

```
// Nested.svelte
<script>
	export let answer;
</script>

<p>The answer is {answer}</p>
```

##### 事件
https://www.sveltejs.cn/tutorial/component-events
```
// index.svelte
<script>
	import Inner from './Inner.svelte';

	function handleMessage(event) {
		alert(event.detail.text);
	}
</script>

<Inner on:message={handleMessage}/>
```
```
// Inner.svelte
<script>
	import { createEventDispatcher } from 'svelte';

	const dispatch = createEventDispatcher();

	function sayHello() {
		dispatch('message', {
			text: 'Hello!'
		});
	}
</script>

<button on:click={sayHello}>
	Click to say hello
</button>
```
##### 绑定
https://www.sveltejs.cn/tutorial/text-inputs
 ```
<script>
	let name = 'world';
</script>

<input bind:value={name}>

<h1>Hello {name}!</h1>
```

##### 生命周期
https://www.sveltejs.cn/tutorial/onmount
https://www.sveltejs.cn/tutorial/ondestroy
https://www.sveltejs.cn/tutorial/update
https://www.sveltejs.cn/tutorial/tick

### 路由
不自带路由方案，可以用第三方库，如
https://github.com/EmilTholin/svelte-routing
```
<script>
  import { Router, Route } from "svelte-routing";
  import NavLink from "./components/NavLink.svelte";
  import Home from "./routes/Home.svelte";
  import About from "./routes/About.svelte";
  import Blog from "./routes/Blog.svelte";

  // Used for SSR. A falsy value is ignored by the Router.
  export let url = "";
</script>

<Router url="{url}">
  <nav>
    <NavLink to="/">Home</NavLink>
    <NavLink to="about">About</NavLink>
    <NavLink to="blog">Blog</NavLink>
  </nav>
  <div>
    <Route path="about" component="{About}" />
    <Route path="blog/*" component="{Blog}" />
    <Route path="/" component="{Home}" />
  </div>
</Router>

```
```
// components/NavLink.svelte
<script>
  import { Link } from "svelte-routing";

  export let to = "";

  function getProps({ location, href, isPartiallyCurrent, isCurrent }) {
    const isActive = href === "/" ? isCurrent : isPartiallyCurrent || isCurrent;

    // The object returned here is spread on the anchor element's attributes
    if (isActive) {
      return { class: "active" };
    }
    return {};
  }
</script>

<Link to="{to}" getProps="{getProps}">
  <slot />
</Link>

```

### 优点
· 代码量少
· 无需diff
· 反应快

### 不足
· 相关生态发展不充分（脚手架，UI库，状态管理）