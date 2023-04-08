# cookie

- **定义**

  HTTP协议本身是无状态的。一次会话，多次交互时（多次向服务器发送消息）服务器端无法判断当前的状态（例如：用户是否登录，用户身份，用户权限等）。为了解决这个问题， 客户端向服务器发起请求，如果服务器需要记录该用户状态，就使用response创建一个Cookie，（在响应头里面添加一个`Set-Cookie`选项），浏览器收到响应后会保存下`Cookie`，之后对该服务器每一次请求中都通过`Cookie`请求头部将`Cookie`信息发送给服务器。（http协议的规范，不管你这次需不需要cookie，每次都会把Cookie提交给服务器）。服务器检查该Cookie，以此来辨认用户状态。Cookie实际上是一小段的文本信息（key-value格式）。客户端也可以通过js读取，设定cookie，实际上通过该方式，浏览器端可以和服务器端通过cookie来交互数据。

- **有效期限**

  默认为浏览器关闭时，自动删除cookie。

  - Expires -- 设置cookie过期时间。
  - Max-Age -- 默认为-1，也就是浏览器关闭时，自动删除cookie，也可以设置为一个正数（1周，1月，1年）

- **作用域**

  - Domain -- Cookie是不可以跨域名的.正常情况下，同一个一级域名下的两个二级域名也不能交互使用Cookie，比如test1.aaa.com和test2.aaa.com，因为二者的域名不完全相同。如果设置Cookie的domain参数为.aaa.com，这样使用test1.aaa.com和test2.aaa.com就能访问同一个cookie
  - Path -- path属性决定允许访问Cookie的路径。比如，设置为"/"表示允许所有路径都可以使用Cookie

- **安全性**

  - Secure -- 如果设置了这个属性，那么只会通过`HTTPS`连接将加密过cookie发送给服务端。
  - HttpOnly -- 为避免跨域脚本 **(XSS)** 攻击，通过**JavaScript**的 `Document.cookie` **API**无法访问带有 `HttpOnly` 标记的`Cookie`，它们只应该发送给服务端。
  - `SameSite Cookie`允许服务器要求某个`cookie`在跨站请求时不会被发送，从而可以阻止跨站请求伪造攻击`（CSRF）`。

  - 防止`cookie`是明文，可以在服务器端利用密钥加密cookie，hash cookie内容

- **存储**

  - 默认浏览器会存储在内存中，如果设置了Expires，Max-Age会保存在文件中
  - 每个`Cookie`长度不能超过**4KB**
  - 浏览器对`Cookie`数量有限制每个域名(Domain)下**FF:最多50个** **Chrome和safari没有硬性限制**

  

- **是否自动于服务器通信**

  每一次请求中浏览器都会自动将`Cookie`信息发送给服务器， 以无形中增加了流量

- **应用场景**

  - 会话状态管理（如用户登录状态、购物车、游戏分数或其它需要记录的信息）
  - 个性化设置（如用户自定义设置、主题等）
  - 浏览器行为跟踪（如跟踪分析用户行为等）

  

- 注意点

  用户可以通过浏览器的设置，来禁用cookie。

  

  

# session

- **定义** 

  session是什么？`Session`是一种记录客户状态的机制，不同于`Cookie`的是`Cookie`保存在客户端浏览器中，而Session保存在服务器上。避免了在客户端`Cookie`中存储敏感数据。

   创建`Session`的同时，服务器会为该`Session`生成唯一的`session id`。

   这个`session id`可以保存在cookie中，在随后的请求中会被发送到服务器端用来获得`Session`情报。

  如果用户禁用了cookie，可以利用URL地址重写，实质上 是通过向 URL 连接添加参数，并把 session ID 作为值包含在连接中。当客户端再次发送请求的时候，会将这个`session id`带上， 服务器接受到请求之后就会依据`session id`找到相应的`Session`，从而再次使用`Session`。

  然而URL地址重写，需要为你的 servlet 响应部分的每个连接添加 session ID.  如何实现?

  Java实现
    把 session ID 加到一个连接可以使用一对方法来简化：response.encodeURL() 使 URL包含 session ID，如果你需要使用重定向，可以使用 response.encodeRedirectURL ()来对 URL 进行编码。encodeURL () 及 encodeRedirectedURL () 方法首先判断 cookie是否被浏览器支持；如果支持，则参数 URL 被原样返回，session ID 将通过 cookies 来维持。
    代码示例:
    不使用Url重写:<a href=http://wwww.myserver.com/servelet/user;userName=awaysrain>Link</a>
    使用Url重写:通过HttpServletResponse接口中的encodeURL()方法编码.
    String myURL = response.encodeURL(http://wwww.myserver.com/servelet/user);
    <a href= <%=myURL%> >Link</a>

  也可以通过struts等框架来实现。

- **有效期限**

  可以通过设置web服务器的session timeout参数来设置有效期限，也可以存储到数据库中，文件中，永久持续化。

- **安全性**

  因为数据是保存在服务器上，比较安全。

- **存储**

  是保存在服务器上。避免了在客户端中存储敏感数据。也可以存储到数据库中，文件中。

- **是否自动于服务器通信**

  只需要传送session id

- **应用场景**

  会话状态管理（如用户登录状态、购物车、游戏分数或其它需要记录的信息）



# localStorage

- **定义** 

  是本地存储，为每一个给定的源（given origin）维持一个独立的存储区域，存储在浏览器客户端。提供一种在cookie之外存储会话数据的路径，提供一种存储大量跨会话存在的数据的机制

  

- **作用域**

  不同浏览器无法共享localStorage或sessionStorage中的信息。相同浏览器的不同页面间可以共享相同的 localStorage（页面属于相同域名和端口）

- **有效期限**

  localStorage生命周期是永久，这意味着除非用户显示在浏览器提供的UI上清除localStorage信息，否则这些信息将永远存在。

  

- **安全性**

  只有相同域名的页面才能互相读取 localStorage，同源策略与 cookie 一致

  WebStorage不会随着HTTP header发送到服务器端，所以安全性相对于cookie来说比较高一些，不会担心截获，但是仍然存在伪造问题

- **存储**

  存放数据大小为一般为5MB,而且它仅在客户端（即浏览器）中保存。

  localStorage和sessionStorage==只能存储字符串类型==，对于复杂的对象可以使用ECMAScript提供的JSON对象的`stringify`和`parse`来处理

- **是否自动于服务器通信**

  不自动于服务器通信

- **应用场景**

  - localStoragese：常用于长期登录（+判断用户是否已登录），适合长期保存在本地的数据，
  - sessionStorage：敏感账号一次性登录

- 使用方法

  ```js
  // get localStorage
  window.localStorage;
  //get sessionStorage
  window.sessionStorage;
  
  //存
      var obj = {"name":"xiaoming","age":"16"}
      localStorage.setItem("userInfo",JSON.stringify(obj));
  
  //取
      var user = JSON.parse(localStorage.getItem("userInfo")
                                                 
  //删除
      localStorage.remove("userInfo);
                                                 
  //清空
      localStorage.clear();
  
  ```

  

# sessionStorage

- **定义** 

  sessionStorage 的所有性质基本上与 localStorage 一致，唯一的不同区别在于：

  sessionStorage 的有效期是页面会话持续，并且重新加载或恢复页面仍会保持原来的页面会话。如果页面会话（session）结束（关闭窗口或标签页），sessionStorage 就会消失。而 localStorage 则会一直存在。**在新标签或窗口打开一个页面时会在顶级浏览上下文中初始化一个新的会话**，这点和 session cookies 的运行方式不同。

  