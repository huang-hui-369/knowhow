# encodeURIComponent

例如下记，要在url上加redirect_url参数，必须用encodeURIComponent，而不是encodeURI

也就是说当你有以下url时redirect_url=的后面部分一定要用encodeURIComponent，它会将：，/,?,&等特殊字符转换成%3A%2F等。

[http://yyy/login?id=1&userType=manager&redirect_url=http://xxx/bukken_detail?bukken=1111&bukennType=1]

一定要encodeURIComponent（http://xxx/bukken_detail?bukken=1111&bukennType=1）这样会变成

[http://yyy/login?id=1&userType=manager&redirect_url=http%3A%2F%2Fxxx%2Fbukken_detail%3Fbukken%3d1111%26bukennType%3d1]

```js
bukkenUrl = "http://xxx/bukken_detail?bukken=1111&bukennType=1"
loginUrl = "http://yyy/login?id=1&userType=manager" + "&redirect_url=" + encodeURIComponent(bukkenUrl);
```

