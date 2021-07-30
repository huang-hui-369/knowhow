# 使用webview

webviewの作用

* 可以显示Html
* 可以支持JS
* 装载页面或resource时有callback event
* JS和Dart的桥接

**优点**
*   修改webpage时，不需要修改app程序，也不需要再更新发布
*   效率比较高
  
**缺点**
*   需要快速响应的设计
*   IOS保持不了cookie

# 选用官方插件webview_flutter
现在常用插件主要有官方插件webview_flutter和flutter_webview_plugin，那各有什么优缺点呢。
  
  **flutter_webview_plugin**
  * flutter_webview_plugin其实是对原生webview控件的封装，性能上接近原生，提供的参数，方法功能多。提供了zoom，localstore，根据class，id提供scroll功能，对js支持比较好
  * 现在比较稳定，bug少
  * Scaffold単位の生成，总是在最顶层，会覆盖其他的ui。
  * 不是官方提供的，有可能不支持最新的flutter版本
  * 在webview上pop一个子画面时，父画面的Webview就会被dispose调。
    例，webview的page1迁移到webview的page2，这时将page2 pop时，page1就显示不了了
    。

  **webview_flutter** 
   * 官方提供的插件，和flutter结合的比较好，webview_flutter只是一个widget，可以嵌入到其他widget。
   * JS和Dart的桥接时用javascriptChannels
   * 装载页面或resource时提供了以下callback
    onWebViewCreated
　  navigationDelegate
　  onPageStarted
　  onPageFinished
   * 现在提供的功能不是很多。

**综上所述尽量用官方插件webview_flutter，或者两者都用，好像不冲突**


# 导入webview_flutter插件

1. pubspec.yaml 文件中追加以下依赖
    ```dart
    dependencies:
    webview_flutter: ^1.0.3
    ```
2. 获取package
   在项目根目录执行
   `flutter pub get`

3. Import it
   `import 'package:webview_flutter/webview_flutter.dart';`
  
4. 针对ios设置
   打开ios/Runner/Info.plist文件追加以下，其中NSAppTransportSecurity节点是为了支持http协议

    ```dart
    <key>io.flutter.embedded_views_preview</key>
    <true/>
    ```
    以下设置是否需要
    ```dart
    <key>io.flutter.embedded_views_preview</key>
    <true/>
    <key>NSAppTransportSecurity</key>
    <dict>
        <key>NSAllowsArbitraryLoads</key>
        <true/>
        <key>NSAllowsArbitraryLoadsInWebContent</key>
    </dict>
     ```

    ![](img\2020-10-16-11-44-18.png)

# 使用Hybrid Composition
当使用keybord输入法时，建议用Hybrid Composition，需要最少API level 19以上。

用法
在组件的in initState()方法中加入以下语句
`WebView.platform = SurfaceAndroidWebView(); `
例
 ```dart
 import 'dart:io';

import 'package:webview_flutter/webview_flutter.dart';

class WebViewExample extends StatefulWidget {
  @override
  void initState() {
    super.initState();
    if (Platform.isAndroid) WebView.platform = SurfaceAndroidWebView();
  }

  @override
  Widget build(BuildContext context) {
    return WebView(
      initialUrl: 'https://flutter.dev',
    );
  }
}
 ```
# Flutter调用JS

1. WebView提供的方法

    WebView创建完成时调用
    onWebViewCreated,

    要显示的url
    initialUrl

    JS执行模式 默认是不调用
    javascriptMode = JavascriptMode.disabled

    使用javascriptChannel JS可以调用Flutter
    javascriptChannels

    拦截请求
    navigationDelegate

    手势
    gestureRecognizers

    页面加载完成
    onPageFinished

    通过WebViewController调用evaluateJavascript方法，即可调用JS。

2. Flutter调用JS的流程：

    2.1 WebView创建完成时在onWebViewCreated 中，获取到WebViewController实例。
    2.2 当页面加载完成之后，即onPageFinished之后，通过WebViewController的evaluateJavascript方法调用JS。
    2.3 evaluateJavascript返回的是Future<String>，通过Future可以获取到JS的返回结果。

    示例代码： 在页面加载完成之后，获取网页标题，并显示在导航栏上。
     ```dart
    WebView(
        initialUrl: "https://flutterchina.club/",
        //JS执行模式 是否允许JS执行
        javascriptMode: JavascriptMode.unrestricted,
        onWebViewCreated: (controller) {
        _controller = controller;
        },
        onPageFinished: (url) {
            _controller.evaluateJavascript("document.title").then((result){
            setState(() {
                _title = result;
            });
            }
        );
        },
    )
     ```
3. JS调用Flutter
    **方法一 使用javascriptChannels**

    javascriptChannels类似于往JS中注册方法，这些方法名要和web端约定好。

    javascriptChannels参数接受Set<JavascriptChannel>一个成员类型为JavascriptChannel的Set集合。

    来看一下JavascriptChannel的用法：

    ```dart
    JavascriptChannel(
        name: "share",
        onMessageReceived: (JavascriptMessage message)    {
                print("参数为： ${message.message}");
        }
    )
    ```

    在JS端就可以调用share方法，同时可以传递参数，在Flutter中通过JavascriptMessage可以获取到参数。

    **JS获取Flutter的返回值**

    js调用Flutter时，除了传递业务需要的参数之外，再添加一个callbackname参数
    通过callbackname参数把JS调用Flutter和Flutter返回参数绑定起来。
    ```dart
    JavascriptChannel(
       name: "share",
       onMessageReceived: (JavascriptMessage message) {
       print("参数： ${message.message}");
                
        String callbackname = message.message; //实际应用中要通过map通过key获取
                  
        String data = "收到消息调用了";
        String script = "$callbackname($data)";
         _controller.evaluateJavascript(script);
       }
    )
    ```

    

    **方法二 使用navigationDelegate调用flutter页面**

    通过约定好的链接myapp://app1，拦截到指定链接后可跳转到原生页面。

    ```dart
    navigationDelegate: (NavigationRequest request) {
    //对于需要拦截的操作 做判断
    if(request.url.startsWith("myapp://")) {
        print("即将打开 ${request.url}");
        //做拦截处理
        //pushto.... 
        
        Application.push(context, request.url);
        return NavigationDecision.prevent;
    }
    
    //不需要拦截的操作
    return NavigationDecision.navigate;
    } ,
    ```


## ios web view 字显示的小
在html中追加缩放为1
```
<head><meta name="viewport" content="width=device-width, initial-scale=1.0"></head>
```
