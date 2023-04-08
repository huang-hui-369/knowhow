## [语法](https://developer.mozilla.org/zh-CN/docs/Web/API/URLSearchParams/URLSearchParams#语法)

```js
var URLSearchParams = new URLSearchParams(init);
```



### [参数](https://developer.mozilla.org/zh-CN/docs/Web/API/URLSearchParams/URLSearchParams#参数)

- *`init`* 可选

  一个 [`USVString`](https://developer.mozilla.org/zh-CN/docs/conflicting/Web/JavaScript/Reference/Global_Objects/String_9094f63a1f7efd350dd69d6a8ae174fb) 实例，一个 [`URLSearchParams`](https://developer.mozilla.org/zh-CN/docs/Web/API/URLSearchParams) 实例，一个 [`USVString`](https://developer.mozilla.org/zh-CN/docs/conflicting/Web/JavaScript/Reference/Global_Objects/String_9094f63a1f7efd350dd69d6a8ae174fb)，或者一个包含 [`USVString`](https://developer.mozilla.org/zh-CN/docs/conflicting/Web/JavaScript/Reference/Global_Objects/String_9094f63a1f7efd350dd69d6a8ae174fb) 的记录。[`USVString`](https://developer.mozilla.org/zh-CN/docs/conflicting/Web/JavaScript/Reference/Global_Objects/String_9094f63a1f7efd350dd69d6a8ae174fb)可以为字符串，二维数组，也可以为对象。

### [返回值](https://developer.mozilla.org/zh-CN/docs/Web/API/URLSearchParams/URLSearchParams#返回值)

一个 [`URLSearchParams`](https://developer.mozilla.org/zh-CN/docs/Web/API/URLSearchParams) 实例。

## [例子](https://developer.mozilla.org/zh-CN/docs/Web/API/URLSearchParams/URLSearchParams#例子)

下面的例子展示了用一个 URL 字符串创建一个 [`URLSearchParams`](https://developer.mozilla.org/zh-CN/docs/Web/API/URLSearchParams) 对象。

```js
// Pass in a string literal
var url = new URL('https://example.com?foo=1&bar=2'); // URL对象
// Retrieve from window.location
var url2 = new URL(window.location);

// Retrieve params via url.search, passed into ctor
var params = new URLSearchParams(url.search); // foo=1&bar=2
var params2 = new URLSearchParams(url2.search);

// Pass in a sequence
var params3 = new URLSearchParams([["foo", 1],["bar", 2]]); 

// Pass in a record
var params4 = new URLSearchParams({"foo" : 1 , "bar" : 2});

// get query parameter
if(params4.has("foo" ) === true) {
    console.log(params4.get(key);)  // 1
}
```



