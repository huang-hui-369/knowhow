# 底部导航栏
android和CupertinoTabView的底部导航栏设计理念不一样。

两者在底部导航栏的tab按钮上基本一样，就是通过tab index来实现tab画面迁移。但是迁移到tab A 页面后，再点击A页面的link，再迁移到A-2画面时，就不一样了。
在Material的Scaffold的bottomNavigationBar框架下，迁移到A-2画面时底部的导航栏就没有了。


![](img\2020-10-06-10-48-39.png)

![](img\2020-10-06-10-49-12.png)

ios 风格的框架底部的导航栏就可以一直保留。

MaterialApp包含了APP中的top level 或者全局的Navigator对象

# 解决方案一(有部分问题)
用nest navagator
可以解决底部bottonNavbar一直在底部显示的问题，但是如果按系统返回键，不能返回上一个页面，就直接退出系统了。

![](img\2020-10-06-11-29-17.png)
按下检索按钮，迁移到详细画面。
![](img\2020-10-06-11-30-25.png)
按下app bar的返回键，很好的返回到list画面

![](img\2020-10-06-11-32-44.png)

按下系统的返回键，就不行了，直接退出程序。
因为在顶级的MaterialApp中包含了一个顶级的navgator,存在于MaterialApp中的Context中
当点击tab b时，在tab b的context里创建了一个tab b的nest navagator，所以点击画面的后退键，会正确响应，但是按下系统的后退键，会调用MaterialApp中的navgator，这个navgator中并没与push 路由，所有会直接退出程序。



# 解决方案二

##　解决思路
用多个navgator，每一个tab对应一个navgator来对应，在tab切换时，切换不同的navgator。
是用Statck，或者IndexStack来控制显示那个tab首页面和用哪个navgator。
MaterialApp 是根据组件的navigatorKey来使用navgator，当组件的navigatorKey变了的话，就用这个组件的navigator来导航，所以拥有一个navigatorKey的组件就都用一个navigator来导航。

其实这种方法还是要拦截返回键处理，在其中加入如下代码才好使
` _navigatorKeys[_currentTab].currentState.maybePop();`
canpopは効きがない
` _navigatorKeys[_currentTab].currentState.canPop();`

### 課題

上記の考えを従って、nest navagatorを利用して、バックアップボタン押下後、nest navagatorの取得ができれば、なんとかできるかな

GlobalKey<NavigatorState>() ?? 