# dialog

dialog.dart

## 显示dialog
可以调用提供的showDialog函数
```dart
Future<T> showDialog<T>({
  @required BuildContext context,
  WidgetBuilder builder,
  bool barrierDismissible = true,
  Color barrierColor,
  bool useSafeArea = true,
  bool useRootNavigator = true,
  RouteSettings routeSettings,
  @Deprecated(
    'Instead of using the "child" argument, return the child from a closure '
    'provided to the "builder" argument. This will ensure that the BuildContext '
    'is appropriate for widgets built in the dialog. '
    'This feature was deprecated after v0.2.3.'
  )
  Widget child,
})
```
### 示例
```dart
static Future<bool> showDeleteConfirmDialog(BuildContext context, String content) {
    return showDialog<bool>(
      context: context,
      builder: (context) {
        return AlertDialog(
            contentTextStyle: Theme.of(context).textTheme.subtitle1.copyWith(
                color: Colors.red,
            ),
          content: Text(content),
          actions: <Widget>[
            FlatButton(
              textColor: Theme.of(context).colorScheme.onPrimary,
              child: Text("キャンセル"),
              // 关闭对话框并返回false
              onPressed: () => Navigator.of(context).pop(false),
            ),
            FlatButton(
              textColor: Theme.of(context).colorScheme.onPrimary,
              child: Text("確認"),
              onPressed: () {
                //关闭对话框并返回true
                Navigator.of(context).pop(true);
              },
            ),
          ],
        );
      },
    );
  }
```

只需要提供builder函数。

## 系统提供了AlertDialog

* 不带title的
![](img\2020-11-27-10-27-56.png)

* 带title的
![](img\2020-11-27-10-32-21.png)

主要用到的有三个属性
1. Widget title
   通常为Text组件默认为headline6
2. Widget content
   通常为Text组件默认为subtitle1
3. List\<Widget\> actions
   通常为FlatButton组件，父类为ButtonBar 

## 系统提供了SimpleDialog

![](img\2020-11-27-10-52-12.png)

主要用到的有两个属性
1. Widget title
2. List\<Widget\> children 通常为纵向排列的Item
   每一个item为SimpleDialogOption对象
   包含了child为Widget和onPressed函数

## SimpleDialogOption
主要提供一个点击的函数。
* final VoidCallback onPressed;
* final Widget child;

## 问题
dialog基本显示在中间，怎么改变显示位置，比如点击button的下方？？？   
