- **模型（Model）** 用于封装与应用程序的业务逻辑相关的数据以及对数据的处理方法。“ Model ”有对数据直接访问的权力，例如对数据库的访问。“Model”不依赖“View”和“Controller”，也就是说， Model 不关心它会被如何显示或是如何被操作。但是 Model 中数据的变化一般会通过一种刷新机制被公布。为了实现这种机制，那些用于监视此 Model 的 View 必须事先在此 Model 上注册，从而，View 可以了解在数据 Model 上发生的改变。（比如：[观察者模式](https://zh.wikipedia.org/wiki/观察者模式)）
- **视图（View）**能够实现数据有目的的显示（理论上，这不是必需的）。在 View 中一般没有程序上的逻辑。为了实现 View 上的刷新功能，View 需要访问它监视的数据模型（Model），因此应该事先在被它监视的数据那里注册。
- **控制器（Controller）**起到不同层面间的组织作用，用于控制应用程序的流程。它处理事件并作出响应。“事件”包括用户的行为和数据 Model 上的改变。



MVC模式在概念上强调 Model, View, Controller 的分离，各個模組也遵循著由 Controller 來處理訊息，Model 掌管資料來源，View 負責資料顯示的職責分離原則，因此在實作上，MVC 模式的 Framework 通常會將 MVC 三個部份分離實作：

- Model 負責資料存取，較現代的 Framework 都會建議使用獨立的資料物件 (DTO, POCO, POJO 等) 來替代弱型別的集合物件。資料存取的程式碼會使用 Data Access 的程式碼或是 ORM-based Framework，也可以進一步使用 Repository Pattern 與 Unit of Works Pattern 來切割資料來源的相依性。
- Controller 負責處理訊息，較高階的 Framework 會有一個預設的實作來作為 Controller 的基礎，例如 Spring 的 DispatcherServlet 或是 ASP.NET MVC 的 Controller 等，在職責分離原則的基礎上，每個 Controller 負責的部份不同，因此會將各個 Controller 切割成不同的檔案以利維護。
- View 負責顯示資料，這個部份多為前端應用，而 Controller 會有一個機制將處理的結果 (可能是 Model, 集合或是狀態等) 交給 View，然後由 View 來決定怎麼顯示。例如 Spring Framework 使用 JSP 或相應技術，ASP.NET MVC 則使用 Razor 處理資料的顯示。

这里有一个通过 [JavaScript](https://zh.wikipedia.org/wiki/JavaScript) 所实现的基于 MVC 模型，需要注意的是：MVC 不是一种技术，而是一种理念。

```
/** 模拟 Model, View, Controller */
var M = {}, V = {}, C = {};

/** Model 负责存放资料 */
M.data = "hello world";

/** View 负责将资料输出给用户 */
V.render = (M) => { alert(M.data); }

/** Controller 作为连接 M 和 V 的桥梁 */
C.handleOnload = () => { V.render(M); }

/** 在网页读取的时候呼叫 Controller */
window.onload = C.handleOnload;
```

在这个简短的程序中就是一个完整的 MVC 模式。