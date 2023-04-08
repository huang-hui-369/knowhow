# 组件

petite-vue的组件有点聊胜于无的意思，通过函数来创建，基本上都是在一个html中用。

也可以通过js，html的引入来使用已经定义好的组件。

- 不带模板的组件

  下记Counter组件，只是定义了组件需要的响应式的数据结构，和方法而已

  ```html
  <script type="module">
    import { createApp } from 'https://unpkg.com/petite-vue?module'
    /* Counter组件定义 */
    function Counter(props) {
      return {
        count: props.initialCount,
        inc() {
          this.count++
        },
        mounted() {
          console.log(`I'm mounted!`)
        }
      }
    }
  
    createApp({
      Counter
    }).mount()
  </script>
  
  <!-- Counter组件在v-scope中引用, 实际上组件的html，css都是在这里定义-->
  <div v-scope="Counter({ initialCount: 1 })" @vue:mounted="mounted">
    <p>{{ count }}</p>
    <button @click="inc">increment</button>
  </div>
  
  <div v-scope="Counter({ initialCount: 2 })">
    <p>{{ count }}</p>
    <button @click="inc">increment</button>
  </div>
  ```

  

- 带模板的组件

  ```html
  <script type="module">
    import { createApp } from 'https://unpkg.com/petite-vue?module'
  
    /* Counter组件定义 */
    function Counter(props) {
      return {
        $template: '#counter-template',  // 这里定义组件的html
        /* $template: `                  // 也可以通过字符串的形式来定义组件的html
                       My count is {{count}}
                       <button @click="inc">+aaa+</button>
                       `,
        */
          
        count: props.initialCount,
        inc() {
          this.count++
        }
      }
    }
  
    createApp({
      Counter
    }).mount()
  </script>
  
  <template id="counter-template">
    My count is {{ count }}
    <button @click="inc">++</button>
  </template>
  
  <!-- reuse it -->
  <div v-scope="Counter({ initialCount: 1 })"></div>
  <div v-scope="Counter({ initialCount: 2 })"></div>
  ```

- tree组件

  ```vue
  <script type="module">
    // import { createApp } from '../src'
    import { createApp } from 'https://unpkg.com/petite-vue?module'
  
     // 1. tree 数据定义
    const treeData = {
      name: 'My Tree',
      children: [
        { name: 'hello' },
        { name: 'wat' },
        {
          name: 'child folder',
          children: [
            {
              name: 'child folder',
              children: [{ name: 'hello' }, { name: 'wat' }]
            },
            { name: 'hello' },
            { name: 'wat' },
            {
              name: 'child folder',
              children: [{ name: 'hello' }, { name: 'wat' }]
            }
          ]
        }
      ]
    }
  
    // 2. tree 组件定义
    function TreeItem(model) {
      return {
        $template: '#item-template',
        model,
        open: false,
        get isFolder() {
          return model.children && model.children.length
        },
        toggle() {
          if (this.isFolder) {
            this.open = !this.open
          }
        },
        changeType() {
          if (!this.isFolder) {
            model.children = []
            this.addChild()
            this.open = true
          }
        },
        addChild() {
          model.children.push({
            name: 'new stuff'
          })
        }
      }
    }
  // 3. create app 只有传递给App的数据，才可以直接在template中引用（v-scope="TreeItem(treeData)"）
    createApp({
      TreeItem,
      treeData
    }).mount()
  </script>
  
  <template id="item-template">
    <div :class="{ bold: isFolder }" @click="toggle" @dblclick="changeType">
      <span>{{ model.name }}</span>
      <span v-if="isFolder">[{{open ? '-' : '+'}}]</span>
    </div>
    <ul v-show="open" v-if="isFolder">
      <li v-for="model in model.children" v-scope="TreeItem(model)"></li>
      <li class="add" @click="addChild">+</li>
    </ul>
  </template>
  
  <p>Double click an item to turn it into a folder.</p>
  <ul v-scope>
    <li class="item" v-scope="TreeItem(treeData)"></li>
  </ul>
  ```

  

# 思考

因为没有webpack，npm等工具来打包编译，build js文件，petite-vue的组件不支持单文件.vue的形式。基本上petite-vue的组件是以html的形式存在的。那可不可以在主html文件中引入其他组件html呢。

例如有下记组件，在page1.html中引入component1.html，和component2.html来使用component1，component2

- component1.html
- component2.html
- component3.html

## 解决办法（没有实际验证）

实际上只要在一个html中引入另一个html就可以了

### 方法1 html5在head中直接引入

```html
<head>
  <link rel="import" href="/path/to/imports/stuff.html">
</head>
```



### 方法2 通过jquery的load（ajax来请求服务器的html的方式来引用）

需要注意的是这种方式，必须发布到web服务器上来引用。

（chrome的ajax不能通过[file:///d:/a.html]方式来引用，  firefox可以引用）

```html
<html> 
  <head> 
    <script src="jquery.js"></script> 
    <script> 
        $(function () {
          var includes = $('[data-include]')
          $.each(includes, function () {
            var file = 'views/' + $(this).data('include') + '.html'
            $(this).load(file)
          })
        })
    </script> 
  </head> 

  <body> 
     <div data-include="header"></div>
	 <div data-include="footer"></div>
  </body> 
</html>

```

### 方法3 将整个html写入到js中，然后通过引入js来实现

a.html中引入b.js

```html
<html> 
  <body>
  <h1>Put your HTML content before insertion of b.js.</h1>
      ...

  <script src="b.js"></script>

      ...

  <p>And whatever content you want afterwards.</p>
  </body>
</html>
```

b.js

```js
document.write(`

    <h1>Add your HTML code here</h1>

     <p>Notice, you do not have to escape LF's with a '\',
        like demonstrated in the above code listing.
    </p>

`);
```

