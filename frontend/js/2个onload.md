# window.onloadを２つ以上使いたい場合問題

如果有两个js文件，都用到windows.onload函数，如果有多个onload函数,只有最后一个加载的会被执行。

```js
window.onload = function(){
  console.log('ロースを注文しました');
};
 
window.onload = function(){
  console.log('カルビを注文しました');
}
```

上記の場合、「カルビを注文しました」としか表示されません。ロースは食べられません。

## addEventListenerを使えばOKです

window.onloadを２つ以上使いたい場合は、addEventListenerを使えばOKです。

```js
window.addEventListener('load', function(){
  console.log('ロースを注文しました');
});
 
window.addEventListener('load', function(){
	console.log('カルビを注文しました');
});
```

这样就会输出

「ロースを注文しました」

「カルビを注文しました」