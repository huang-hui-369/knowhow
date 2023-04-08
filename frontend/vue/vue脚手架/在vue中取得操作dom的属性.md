# nextTick

```js
this.$nextTick(() => {
    // 取得dom的属性
    let storeSeach = document.getElementById("storeSeach").offsetTop;
    // 操作window的方法
    window.scrollTo(0,storeSeach - this.topHeight);
});
```