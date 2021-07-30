# 1. 全局捕获
全局的异常常常是一些不可预计的应行时错误。

## 1.1 捕获并上报同步异常
捕获同步异常只需要定义一个FlutterError.onError函数就可以。

比如调用原生代码发生的平台异常,创建widget的异常。
当 release 模式下发生错误时，应用理应立即关闭，可以使用下面的回调方法:
```dart
import 'dart:io';

import 'package:flutter/foundation.dart';
import 'package:flutter/material.dart';

void main() {

  const bool kReleaseMode = const bool.fromEnvironment("dart.vm.product");

  FlutterError.onError = (FlutterErrorDetails details) {
    if (kReleaseMode) {
        /// 交由3. 捕获并上报 Dart异常 统一处理错误信息
        Zone.current.handleUncaughtError(details.exception, details.stack);
        exit(1);
    } else {
        /// debug mode 输出错误信息到console
        FlutterError.dumpErrorToConsole(details);
    }
      
  };
  runApp(MyApp());
}

// rest of `flutter create` code...
```
这个回调方法也可以上报错误到日志服务平台

## 1.2. 自定义错误页面，重写 ErrorWidget.builder
创建自定义的错误页面只需要定义一个ErrorWidget.builder函数。

```dart
void main() {
  ErrorWidget.builder = (FlutterErrorDetails flutterErrorDetails) {
    print(flutterErrorDetails.toString());
    return Center(
      child: Text("Flutter 崩溃了"),
    );
  };
  runApp(MyApp());
}
```

# 1.3. 捕获并上报异步异常
要想捕获异步异常只需要让程序运行在统一个沙箱，将运行的程序包裹在函数
runZonedGuarded<Future<void>>(程序运行函数，错误处理函数)

```dart
runZonedGuarded<Future<void>>(
  /// 程序运行函数
  () async {
    runApp(MyApp());
  }, 
  /// 错误处理函数
  (Object error, StackTrace stackTrace) {
    _reportError(error, stackTrace);
  }
);
```
或者包裹在函数runZoned中
```dart
runZoned(
   /// 程序运行函数
  () {
   runApp(MyApp());
  }, 
  /// 错误处理函数
  onError:  (e) {
    print(e);
  } => 
); 

```

# 2. 捕获局部异常
局部异常常常是可以预计的异常，

* 用户输错信息，需要向客户显示错误信息，让用户修正输入。
* 调用api产生的错误，需要向客户显示错误信息，让用户修正输入。

如果想要捕获异步调用产生的异常一个方法是上计提到过用runZoned的将将运行代码包裹在一个沙箱。
或者每一步都调用await使异步函数变成同步函数。这是因为每一个异步函数的异常都只属于自己的Future，每包裹一层的Future都是不同的。
例如 在button按下函数中，调用异步函数a，a调用异步函数b，b再调用异步函数c，假设c中抛出异常，如果异步函数b中不用 await 调用异步函数c，异步函数a中不用await 调用异步函数b，将异常抛出到函数a的future中，那么button按下函数是捕获不到异步函数c的异常的。

例如如下
api层的代码
```dart
Future<UserInfo> login(String userId,String password) async {
    await httputil.get(userId,password);
  }
 ```
 bl层的代码
```dart
Future<void> login(String userId,String password) async {
    UserInfo user = await api.login(userId,password);
    /// 将user保存到全局变量中
    _user = user;
  }
 ```
widget层的代码
```dart
void loginClick() {
  String userId = 页面个的用户id
  String password = 页面的密码
  bl.login(userId,password).then(
    // 结果ok的话迁移到home画面
    (value) {
        迁移到home画面
    }
  ).catchError((e, s) {
    /// NGの場合、エラーメッセージ表示
    Scaffold.of(context).showSnackBar();
  });
}
```

# 3. 调用http共通异常处理

# 子组件，页面的异常处理
通常UI子组件，UI页面应该自己管理状态，即使数据没有取到，也应该显示一个空的组件不能出现异常，因为在build函数中是没有办法捕获到异常的。

下记的处理只能捕获异步函数抛出的错误。

```dart
runZonedGuarded<Future<void>>(
  /// 程序运行函数
  () async {
    runApp(MyApp());
  }, 
  /// 错误处理函数
  (Object error, StackTrace stackTrace) {
    _reportError(error, stackTrace);
  }
);
```

如果在页面的initState中,像下面这样写，是上面的错误处理函数没有办法捕获到的。
会直接在widget中显示错误。

```
@override
  void initState() {
    MspLogger.logger.d("ExpansionPaneCheckBox init [${widget.selectItemCategoryCode},${widget.value}]");
    _checkedCodes = widget.value;
    _isExpanded = widget.isExpanded;
    if(!_selectCategoriesBl.haveData()) {
      throw ErrorMessage.get_select_items_error;
    }
    super.initState();
  }
```

如果在页面的initState中,像下面写成异步函数，就会被上面的错误处理函数捕获到了。

```
 @override
  void initState() {
    MspLogger.logger.d("ExpansionPaneCheckBox init [${widget.selectItemCategoryCode},${widget.value}]");
    _checkedCodes = widget.value;
    _isExpanded = widget.isExpanded;
    checkError();
    super.initState();
  }

  Future<void> checkError() async {
    if(!_selectCategoriesBl.haveData()) {
      throw ErrorMessage.get_select_items_error;
    }
  }
```