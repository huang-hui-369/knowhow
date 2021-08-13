# 概述

就是利用webview中的navigationDelegate(url) call back事件处理，当用户点击link时，会调用navigationDelegate方法，然后在方法中调用url_launcher的launch(url)方法,就可以跳转到手机中默认的browser。

# 用到的组件

- webview_flutter 2.0.2
- url_launcher 6.0.3

# 一个简单的例子

```dart

import 'package:url_launcher/url_launcher.dart';
import 'package:webview_flutter/webview_flutter.dart';

class MyApp extends StatelessWidget {
  // This widget is the root of your application.
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
        title: 'Title Your App',
        theme: ThemeData(
          primarySwatch: Colors.blue,
        ),
        home: Scaffold(
            body: Container(
          child: WebView(
            initialUrl: 'https://website.com',
            javascriptMode: JavascriptMode.unrestricted,
            onWebViewCreated: (WebViewController webViewController) {
              _controller.complete(webViewController);
            },
            /// 用户点击webview中的link时会调用此callback方法
            navigationDelegate: (NavigationRequest request) {
              if (request.url.startsWith("https://website.com")) {
                return NavigationDecision.navigate;
              } else {
                /// 迁移到手机浏览器中打开url 
                _launchURL(request.url);
                return NavigationDecision.prevent;
              }
            },
          ),
        )));
  }

  _launchURL(String url) async {
    if (await canLaunch(url)) {
      await launch(url);
    } else {
      throw 'Could not launch $url';
    }
  }
}

```

# 完整的例子

```dart
import 'dart:convert';
import 'dart:io';

import 'package:flutter/foundation.dart';
import 'package:flutter/gestures.dart';
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';
import 'package:msp_app/bloc/models/index.dart';
import 'package:msp_app/ui/widgets/msp_app_bar.dart';
import 'package:msp_app/utils/date_util.dart';
import 'package:webview_flutter/platform_interface.dart';
import 'package:webview_flutter/webview_flutter.dart';
import 'package:url_launcher/url_launcher.dart';



// ignore: must_be_immutable
class NotificationDetail extends StatelessWidget {
  
  NotificationBody detail;

  NotificationDetail(this.detail, {Key key}) : super(key: key);

  @override
  Widget build(BuildContext context) {

    print("notification detail[${detail.content}]");

    String createdTime = MspDateFormat.yyyyMMddHHmmSS(detail.createdTime);

    return Scaffold(
      appBar: MspAppBar(
        title: 'お知らせ詳細',
      ),
      body: ContentView(detail.content),

    );
  }
}


class ContentView  extends StatefulWidget {

  /// 要显示在webview中的html字符串
  String htmlStr;

  /// 可以让用户上下左右移动webview
  final Set<Factory<OneSequenceGestureRecognizer>> gestureRecognizers;

  /// 构造函数，需要接受一个要显示在webview中的html字符串参数
  ContentView(this.htmlStr, {this.gestureRecognizers, Key key}) : super(key: key);

  @override
  State createState() => new ContentViewState();
}

class ContentViewState extends State<ContentView> {

  static const  String errorHtmlPath = 'assets/html/error.html';

  WebViewController _controller;

  @override
  void initState() {
    if (Platform.isAndroid) WebView.platform = SurfaceAndroidWebView();
    super.initState();
  }

  @override
  Widget build(BuildContext context) {
    return WebView(
      gestureRecognizers: widget.gestureRecognizers,
      javascriptMode: JavascriptMode.unrestricted,

      onWebViewCreated: (WebViewController webViewController) {
        _controller = webViewController;
        _controller.scrollTo(0, 0);
        /// 初期显示的html字符串
        _loadHtmlStr(this.widget.htmlStr);
      },
      /// 出错的时候，显示错误消息页面
      onWebResourceError: (WebResourceError error) {
        _loadHtmlFromAssets();
      },
      /// 用户点击webview中的link时会调用此callback方法
      navigationDelegate: (NavigationRequest request) {
          _launchURL(request.url);
          return NavigationDecision.prevent;
      },
    );
  }

  /// 迁移到手机浏览器中并打开url 
  _launchURL(String url) async {
    if (await canLaunch(url)) {
      await launch(url);
    } else {
      throw 'Could not launch $url';
    }
  }

  /// 从assert中装载错误消息html页面
  _loadHtmlFromAssets() async {
    String fileHtmlContents = await rootBundle.loadString(errorHtmlPath);
    _controller.loadUrl(Uri.dataFromString(fileHtmlContents,
        mimeType: 'text/html', encoding: Encoding.getByName('utf-8'))
        .toString());
  }

  /// 将html字符串显示到webview中
  void _loadHtmlStr(String htmlStr) async {
    final String contentBase64 =
    base64Encode(const Utf8Encoder().convert(htmlStr));
    await _controller.loadUrl('data:text/html;base64,$contentBase64');
  }

}

```