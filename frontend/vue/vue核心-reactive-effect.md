# vue核心响应式数据

为什么定义的数据变化了会自动发应到页面的dom中，vue3的原理主要有两点

- **reactive()**函数 -- 接收一个对象作为参数，并返回一个JS Proxy 代理对象。
- **effect()**函数 -- 用于定义副作用，它的参数就是副作用函数

例如 下记代码 

首先通过reactive函数定义了obj对象的Proxy（JS 的功能），这样就可以通过Proxy来监视obj对象并执行响应的代码（副作用函数），当调用obj对象的set，或者get方法时，就可以通过副作用来调用effect()函数，来实现数据变化了会自动发应到页面的dom中。

```js
import { effect, reactive } from '@vue/reactivity'
// 使用 reactive() 函数定义响应式数据
const obj = reactive({ text: 'hello' })
// 使用 effect() 函数定义副作用函数
effect(() => {
     document.body.innerText = obj.text
})

// 一秒后修改响应式数据，这会触发副作用函数重新执行
setTimeout(() => {
  obj.text += ' world'
}, 1000)
```

