# 利用插件 [flutter_cached_pdfview](https://pub.dev/packages/flutter_cached_pdfview)


### 1. Depend on it
Add this to your package's pubspec.yaml file:
```dart
dependencies:
  flutter_cached_pdfview: ^0.3.3
```

### 2. Install it
You can install packages from the command line:

with Flutter:
```dart
$ flutter pub get
```
Alternatively, your editor might support flutter pub get. Check the docs for your editor to learn more.

### 3. Import it
Now in your Dart code, you can use:

```dart
import 'package:flutter_cached_pdfview/flutter_cached_pdfview.dart';
```

### 4. sample

```dart
Container(
    height: 300,
    margin: EdgeInsets.only(top: 20),
    child: PDF(onError: (error) {
        print("pdf error[${error.toString()}]");
    },
        gestureRecognizers:
        <Factory<OneSequenceGestureRecognizer>>[
        new Factory<OneSequenceGestureRecognizer>(() => PanGestureRecognizer(),),].toSet()
    ).fromUrl(
    'https://www.futari-passport.metro.tokyo.lg.jp/doc/%E3%80%90%E5%8D%94%E8%B3%9B%E8%A6%8F%E7%B4%84&%E6%A7%98%E5%BC%8F%E3%80%91%EF%BC%B4%EF%BC%AF%EF%BC%AB%EF%BC%B9%EF%BC%AF%E3%81%B5%E3%81%9F%E3%82%8A%E7%B5%90%E5%A9%9A%E5%BF%9C%E6%8F%B4%E3%83%8F%E3%82%9A%E3%82%B9%E3%83%9B%E3%82%9A%E3%83%BC%E3%83%88%E4%BA%8B%E6%A5%AD%E5%8D%94%E8%B3%9B%E8%A6%8F%E7%B4%84.pdf',
    placeholder: (progress) => Center(child: Text('$progress %')),
    errorWidget: (error) => Center(child: Text(error.toString())),
    ),
            
)
```

### 5 问题
整个页面是一个滚动组件SingleChildScrollView，然后加入PDF，这个时候滚动就会冲突，是响应SingleChildScrollView还是pdf。
