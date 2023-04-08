# 使用 Fetch

[Fetch API](https://developer.mozilla.org/zh-CN/docs/Web/API/Fetch_API) 提供了一个 JavaScript 接口，用于访问和操纵 HTTP 管道的一些具体部分，例如请求和响应。它还提供了一个全局 [`fetch()`](https://developer.mozilla.org/zh-CN/docs/Web/API/fetch) 方法，该方法提供了一种简单，合理的方式来跨网络异步获取资源。

这种功能以前是使用 [`XMLHttpRequest`](https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest) 实现的。Fetch 提供了一个更理想的替代方案，可以很容易地被其他技术使用，例如 [`Service Workers`](https://developer.mozilla.org/zh-CN/docs/Web/API/Service_Worker_API)。Fetch 还提供了专门的逻辑空间来定义其他与 HTTP 相关的概念，例如 [CORS](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/CORS) 和 HTTP 的扩展。

请注意，`fetch` 规范与 `jQuery.ajax()` 主要有以下的不同：

- 当接收到一个代表错误的 HTTP 状态码时，从 `fetch()` 返回的 Promise **不会被标记为 reject**，即使响应的 HTTP 状态码是 404 或 500。相反，它会将 Promise 状态标记为 resolve（如果响应的 HTTP 状态码不在 200 - 299 的范围内，则设置 resolve 返回值的 [`ok`](https://developer.mozilla.org/zh-CN/docs/Web/API/Response/ok) 属性为 false），仅当网络故障时或请求被阻止时，才会标记为 reject。
- `fetch` **不会发送跨域 cookie**，除非你使用了 *credentials* 的[初始化选项](https://developer.mozilla.org/zh-CN/docs/Web/API/fetch#参数)。（自 [2018 年 8 月](https://github.com/whatwg/fetch/pull/585)以后，默认的 credentials 政策变更为 `same-origin`。Firefox 也在 61.0b13 版本中进行了修改）

一个基本的 fetch 请求设置起来很简单。看看下面的代码：

```js
fetch('http://example.com/movies.json')
  .then(response => response.json())
  .then(data => console.log(data));
```



这里我们通过网络获取一个 JSON 文件并将其打印到控制台。最简单的用法是只提供一个参数用来指明想 `fetch()` 到的资源路径，然后返回一个包含响应结果的 promise（一个 [`Response`](https://developer.mozilla.org/zh-CN/docs/Web/API/Response) 对象）。

当然它只是一个 HTTP 响应，而不是真的 JSON。为了获取 JSON 的内容，我们需要使用 [`json()`](https://developer.mozilla.org/zh-CN/docs/Web/API/Response/json) 方法（该方法返回一个将响应 body 解析成 JSON 的 promise）。



其实不是对fetch的封装，而是对下面几方面的处理

1. 对请求域名的统一管理-baseUrl，因为你一个项目至少有dev，test，pre，pro4个环境，也就是4个域名（有些公司都是一个域名，然后通过host来解决）
2. 对请求参数的处理，比如get请求， 参数值为null， undefined，空字符串的可以统一过滤掉
3. 对请求头的统一处理，比如把token带到请求头，参数验签等
4. 对请求响应的处理，比如只有code===200才是正常业务数据，resolve(data)，其他情况直接toast提示错误信息，然后reject(data), 因为业务代码里面一定会出现需要特殊code来下一步处理的情况

# 注意点

- ### 省略return的函数定义方式

  要注意如下函数定义方式，实际上只是省略了return，等价于 return response.json()

  ```js
  .then(response => response.json())
  ```

  但是如下的方式一定要加return。

  ```js
  .then((response) => {
   	return response.json()
   })
  ```

- ### 向服务器发送json数据的format

  注意一定数据的最后不能有多余的逗号，如果有逗号，服务器会返回空数据

  如下的数据OK

  ```json
  {
      'name1': 'value1',
      'name2': 'value2'
  }
  ```

  如下的数据NG，因为多了个逗号，（猜测服务器解析json数据错误，会返回空数据）

  ```json
  {
      'name1': 'value1',
      'name2': 'value2',
  }
  ```

  