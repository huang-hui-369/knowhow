# 在vue中当想鼠标停留在元素上时，改变显示的颜色，内容。
例如：  鼠标不在元素上时，显示向下的三角，当鼠标停留在元素上时，显示三角 和 展开，同时改变字体颜色。

## 解决办法
vue 提供了v-on属性可以绑定浏览器原生的事件，所以只要利用mouseover和mouseLeave事件就可以了。

```html
<div id="app">
        <div class="container">
            <p v-on:mouseover="mouseOverAction" v-on:mouseleave="mouseLeaveAction">{{message}}</p>
            <p v-if="hoverFlag">hoverされました</p>
     　　</div>
    </div>
    <script>
        new Vue({
            el: '#app',
            data:{
                message: 'hoverしてください',
                hoverFlag: false,
            },
            methods: {
                mouseOverAction(){
                    this.hoverFlag = true
                },
                mouseLeaveAction(){
                    this.hoverFlag = false
                }
            }
        })
    </script>
```