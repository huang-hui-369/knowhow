# Promise

从字面上来说就是承诺的意思，古人云：“一诺千金”，这种“承诺将来会执行”的对象在JavaScript中称为Promise对象。

js本身是单线程的（要用来更新UI，响应用户点击输入等操作的主线程），但实际上还有其他线程，如事件触发线程、ajax请求线程等。所以耗时任务都是通过异步方式来执行，（猜测通过浏览器开启线程来执行的）所以需要call back函数。Promise规范了这种异步执行的流程（链式调用）。

通过Promise对象.then(function resolve(data))来承诺保证在耗时任务正常完成情况下，resolve函数（画面UI更新）将被调用。

通过Promise对象.catch(function reject(error))来承诺保证在耗时任务异常情况下reject函数（显示错误消息）将被调用。

注意 resolve函数和reject函数的参数为接受任何类型的单个参数。

> promise对象有两个特点：

- 对象的状态不受外界影响，pending(进行中)、fulfiled(已完成)、rejected(已失败)
- 一旦状态改变，就不会再变，任何时候都可以得到这个结果，状态改变只有两种可能，称为（resolved）

let p = new Promise(参数1)，创造一个实例时，接受一个函数作为参数1；

参数1，作为函数，也接受两个参数分别是resolve和reject。它们是两个函数，由 JavaScript 引擎提供，不用自己部署。**注意：**这两个函数的作用就是改变Promise对象的状态，一个是成功，一个是失败，一旦调用了其中一个，状态就不可改变和逆转；

Promise实例生成以后，可以用then方法分别指定resolved状态和rejected状态的回调函数，即使用p.then()来进一步的操作，**注意：**then()可以接受两个参数，这两个参数也是函数，第一个表示成功的操作，就是调用了resolve(a1)后会进入，第二个表示失败的操作，就是调用了reject(a1)后会进入，但第二个可以省略;

p.catch和.then()第二个参数的效果是一样,都会进入；

# Promise 定义

就是在函数里包一层Promise对象。

正常情况下，会调用传递进来的resolve函数

异常情况下，会调用传递进来的reject函数

```js
fetchjson(url, params) {
    return new Promise((resolve, reject) =>{
        // 网络访问耗时任务
        fetch(url, params)
        .catch((error) => {
            throw new Error("ネットワークエラーが発生しました。");
        })
        .then((res)=>{
            if (res.ok) {
                return res;
            }
            throw new Error("サーバー側はエラーが発生しました。");
        })
        .then((res) => {
            return res.json();
        })
        .catch((error) => {
            console.log(error);
            if(!toast) {
                toast = KBCom.createToast();
                toast.error(error.toString());
            }
        })
        .then(data => {
            if(data.status!=0) {
                console.log(data);
                // 出错时，调用reject函数
                reject(data);
            }
            // 耗时任务正常完成时，调用resolve函数
            resolve(data.data);
        });
    });
}
```

# call Promise

```js
// APIを呼び出す
fetchjson("/api/xxx", {param1:"a",param2:"b"})
	// 正常情况下的处理，resolve函数被调用
    .then((data)=>{
        // login　OKの場合,ユーザー情報をSessionに保存する
        let session = KBTool.Session();
        session.saveLoginUser(data);
        try {
            let redirectUrl = KBTool.UrlParams().getQueryParams("redirect_url");
            // "redirect_url"引数がある場合、"redirect_url"へ遷移する。
            redirectUrl = decodeURIComponent(redirectUrl);
            KBRoute().move(redirectUrl);
        } catch(err) {
            console.log(err);
            // "redirect_url"引数がない場合、ホーム画面へ遷移する。
            KBRoute().move(KBRoute().home);
        }
	})
	// 出错的情况下处理 reject函数被调用
    .catch((data)=>{
        // login　NGの場合
        this.toast.error("APIエラーが発生しました。");
	})
	.finally(()=>{
    	// 不管成功或者出错都会执行
        // 当promise的状态确定时，即无论拿到的是正常结果还是错误信息，总会执行这个finalHandle函数。用.finally可以做一些清理操作，比如发起网络请求后，可以停止loading显示。
});
```

# Promise 执行方式

1. ## 串行方式执行异步任务

   像下面这样顺序执行job1，job2，job3只用每一个任务正常执行完成时再执行下一个Job，出错时，就不执行下一个Job

   ```js
   job1.then(job2).then(job3).catch(handleError);
   ```

2. ## 并行执行异步任务

   像下面这样同时执行p1，p2

   ```js
   Promise.all([p1, p2])
       .then(function (resultArray) { // 获得一个Array: [result1, result2]
           console.log(resultArray[0]); // result1
       	console.log(resultArray[1]); // result2
       })
       .catch((error)=>{
   		console.log(error);
   });
   ```

   

3. ## 竞争方式执行异步任务

   像下面这样同时以竞争的方式执行p1，p2，如果p1执行的快，就返回p1的执行结果，并放弃p2。反之，如果p2执行的快，就返回p2的执行结果，并放弃p1。

   ```
   Promise.race([p1, p2]).then(function (result) {
       console.log(result); // 'P1' or 'P2'
   });
   ```

   

# Promise() 构造器

**`Promise`** 构造器主要用于包装不支持 promise（返回值不是 `Promise`）的函数。

在promise对象创建时，`excutor`会立即执行。一旦`exucutor`执行完，要么产生`value`，要么产生`error`，此时会立即调用`resolve`（当产生`value`时）或者调用`reject`（当产生`error`）时，内部状态也会随之改变

resolve时state为fullfilled，result为data

reject时state为rejecteded，result为error

注意，当`excutor`里面即使调用了多个`resolve`和`reject`，其最终还是只执行一个，其他的都被忽略掉。

```js
let promise = new Promise(function(resolve, reject) {
  resolve("done");

  reject(new Error("…")); // ignored
  setTimeout(() => resolve("…")); // ignored
});
```



```js
const promise1 = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve('foo');
  }, 300);
});

promise1.then((value) => {
  console.log(value);
  // expected output: "foo"
});

console.log(promise1);
// expected output: [object Promise]
```

## [语法](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/Promise#语法)

```js
new Promise(executor)
// executor为一个函数
function(resolveFunc, rejectFunc){
    // 通常是一些异步耗时操作
}
// resolveFunc和rejectFunc定义
function(data) {
    // 通常是UI更新操作
}
```

当该 promise 动态插入[promise 链](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise#promise的链式调用)的情况下，该 resolveFunc的 `data` 参数可能是另一个 promise 对象。

关于 `executor`，需理解以下几点：

- 该 `executor` 的返回值将被忽略，将调用resolveFunc。
- 如果在该 `executor` 中抛出一个错误，该 promise 将调用rejectFunc。

## 例子

为了提供一个拥有 promise 功能的函数，简单的返回一个 promise 即可：

```js
function myAsyncFunction(url) {
  return new Promise((resolve, reject) => {
    const xhr = new XMLHttpRequest()
    xhr.open("GET", url)
    xhr.onload = () => resolve(xhr.responseText)
    xhr.onerror = () => reject(xhr.statusText)
    xhr.send()
  });
}
```



# 总结

- Promise是一个异步函数的 容器
- 如果调用`resolve`函数和`reject`函数时带有参数，那么它们的参数会被传递给then中的回调函数。
-  最好在resolve（）前面加上 return 语句，否则，resolve后面的代码还是会执行。
- then（func1,func2）,  func1等待上方的异步函数的状态变为resolved才会执行，func2会等上方的异步函数变为reject时会执行。

# 参考文档

Promise原理 https://mengera88.github.io/2017/05/18/Promise%E5%8E%9F%E7%90%86%E8%A7%A3%E6%9E%90/