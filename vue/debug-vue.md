# debug vue in chrome

因为vue会将vue文件生成js后在浏览器中显示，所以debug时，会找不到js对应的vue文件。
需要在vue.config.js中设置source-map。

```js
module.exports = {
    configureWebpack: {
        devtool: 'source-map'
    },
}    
```

设置了source-map后就可以在Sources --> webpack://src/...下找到相应的vue文件 
![](img\2021-08-16-19-25-08.png)
