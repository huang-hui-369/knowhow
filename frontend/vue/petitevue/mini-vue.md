# Petite-vue 

## Petite-vue能干什么

Petite-vue可以作为独立的脚本文件使用，无需构建步骤！如果你有一个后端框架（springboot、larabel...），并且它已经渲染了大部分的 HTML，或者你的前端逻辑并不复杂，不需要构建步骤。所以Vue 也提供了另一个适用于此类无构建步骤场景的替代版 petite-vue，主要为渐进式增强已有的 HTML 作了特别的优化。功能更加精简，十分轻量。

1. 大小只有5.8kb
2. Vue 兼容模版语法
3. 基于DOM，就地转换
4. **响应式驱动**

## Petite-vue的引入流程

### 1. 方法一  使用 ES 模块构建，通过默认的v-scope 自动mount

- createApp（obj）中的参数obj为所有表达式的根作用域这个obj的属性，方法都会暴露在v-scope中（在v-scope中可以调用）

  根作用域的count变化的话，会自动反映到v-scope中dom元素（通过表达式{{ count }}来绑定到dom元素）。

- mount（）不传递参数的话，默认会加载到v-scope的dom元素中。

```js
<div v-scope>
    <button @click="decrement">-</button>
    <span>{{ count }}</span>
    <button @click="increment">+</button>
 </div>

  <script type="module">
    import { createApp } from 'https://unpkg.com/petite-vue?module'
    createApp({        // 根作用域对象
      count: 0,
      increment() {
        this.count++
      },
      decrement() {
        this.count--
      },
     }).mount() // 会默认加载到v-scope的dom元素中
  </script>

```

### 2. 方法二 引入Petite-vue

```js
<div v-scope>
    <button @click="decrement">-</button>
    <span>{{ count }}</span>
    <button @click="increment">+</button>
 </div>

<script src="https://unpkg.com/petite-vue"></script>
<script type="text/javascript">
    PetiteVue.createApp({
        count: 0,
        increment() {
            this.count++
        },
        decrement() {
            this.count--
        },
    }).mount();
</script>


```

