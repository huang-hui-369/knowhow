- ## **动态地切换 class**

  如果isActive为true，hasError为true，class 列表将变为 `"static active text-danger"`。

  ```vue
  <div
    class="static"
    v-bind:class="{ active: isActive, 'text-danger': hasError }"
  ></div>
  ```

  

- ## bind classObject

  这样定义给上面的效果一样

  ```vue
  <div v-bind:class="classObject"></div>
  ```

  ```js
  data: {
    classObject: {
      active: true,
      'text-danger': false
    }
  }
  ```

  

- ### [数组语法](https://cn.vuejs.org/v2/guide/class-and-style.html#数组语法)

  ```vue
  <div v-bind:class="[activeClass, errorClass]"></div>
  ```

  也可用三元表达式

  ```vue
  <div v-bind:class="[isActive ? activeClass : '', errorClass]"></div>
  ```

  

  ```js
  data: {
    activeClass: 'active',
    errorClass: 'text-danger'
  }
  ```

  渲染为

  ```html
  <div class="active text-danger"></div>
  ```

  