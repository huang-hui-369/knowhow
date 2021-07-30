# Flutter Drawer 侧边栏

在 Scaffold 组件里面传入 drawer 参数可以定义左侧边栏，传入 endDrawer 可以定义右侧边 栏。侧边栏默认是隐藏的，我们可以通过手指滑动显示侧边栏，也可以通过点击按钮显示侧 边栏。

```dart
return Scaffold(
    appBar: AppBar(
    title: Text("Flutter App"), ),
    drawer: Drawer(
        child: Text('左侧边栏'),
    ),
    endDrawer: Drawer(
        child: Text('右侧侧边栏'), ),
);
```

## DrawerHeader 属性

* decoration 设置顶部背景颜色
* child 配置子元素
* padding 内边距
* margin 外边距

## UserAccountsDrawerHeader

* decoration 设置顶部背景颜色
* accountName 账户名称
* accountEmail 账户邮箱
* currentAccountPicture 用户头像
* otherAccountsPictures 用来设置当前账户其他账户头像
* margin

 