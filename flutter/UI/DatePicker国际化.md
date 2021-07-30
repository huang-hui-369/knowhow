### アプリが対応する言語のロケールを指定する

```dart
import 'package:flutter/material.dart';
import 'package:flutter_localizations/flutter_localizations.dart';

MaterialApp(
 localizationsDelegates: [
   GlobalMaterialLocalizations.delegate,
   GlobalWidgetsLocalizations.delegate,
   GlobalCupertinoLocalizations.delegate,
 ],
 supportedLocales: [
    const Locale('en'),
    const Locale('ja'),
  ],
  // ...
)
```

### showDatePicker() でロケールを設定する

```dart
showDatePicker(
  context: context,
  initialDate: DateTime.now(),
  firstDate: DateTime(DateTime.now().year),
  lastDate: DateTime(DateTime.now().year + 6),
  locale: const Locale('ja'),
);
```

### DatePickerの色を変える

```dart
showDatePicker(
  context: context,
  initialDate: DateTime.now(),
  firstDate: DateTime(DateTime.now().year),
  lastDate: DateTime(DateTime.now().year + 6),
  builder: (BuildContext context, Widget child) {
    return Theme(
      data: ThemeData(
        // ヘッダーの色
        primaryColor: Colors.red,
        // 日付選択部の色
        dialogBackgroundColor: Colors.green,
        // 選択されたときのテキストの色
        accentTextTheme: TextTheme(
          body2: TextStyle(
            color: Colors.black,
          ),
        ),
        // 選択されたときの円の色
        accentColor: Colors.purple,
        // OK/CANCELボタンのテキストの色
        buttonTheme: const ButtonThemeData(
          textTheme: ButtonTextTheme.accent,
        ),
      ),
      child: child,
    );
  },
);
```
### 設定

```dart
final selectedDate = await showDatePicker(
    context: context,
    // 初期選択されている日付
    initialDate: DateTime.now(),
    // 選択可能な最小の日付
    firstDate: DateTime(DateTime.now().year),
    // 選択可能な最大の日付
    lastDate: DateTime(DateTime.now().year + 6),
    // 日付の選択形式
    initialDatePickerMode: DatePickerMode.year,
    // 指定した日を選択できないようにする
    selectableDayPredicate: (DateTime date) =>
        date.weekday == 6 || date.weekday == 7 ? false : true,
    // 日本語化
    // locale: const Locale('ja'),
    locale: LocaleType.jp,
    // DatePickerの色を変える
    builder: (BuildContext context, Widget child) {
    return Theme(
        data: ThemeData(
        // ヘッダーの色
        primaryColor: Colors.red,
        // 日付選択部の色
        dialogBackgroundColor: Colors.green,
        // 選択されたときのテキストの色
        accentTextTheme: TextTheme(
            body2: TextStyle(
            color: Colors.black,
            ),
        ),
        // 選択されたときの円の色
        accentColor: Colors.purple,
        // OK/CANCELボタンのテキストの色
        buttonTheme: const ButtonThemeData(
            textTheme: ButtonTextTheme.accent,
        ),
        ),
        child: child,
    );
    },
);

// キャンセルボタンやダイアログの外をタップしてダイアログを閉じたとき、NULLが返ってくる
if (selectedDate != null) {
    // do something
}

```

###　demo

```dart
// Copyright 2019 The Flutter team. All rights reserved.
// Use of this source code is governed by a BSD-style license that can be
// found in the LICENSE file.

import 'dart:async';

import 'package:flutter/material.dart';
import 'package:flutter_gen/gen_l10n/gallery_localizations.dart';
import 'package:intl/intl.dart';

enum PickerDemoType {
  date,
  time,
}

class PickerDemo extends StatefulWidget {
  const PickerDemo({Key key, this.type}) : super(key: key);

  final PickerDemoType type;

  @override
  _PickerDemoState createState() => _PickerDemoState();
}

class _PickerDemoState extends State<PickerDemo> {
  DateTime _fromDate = DateTime.now();
  TimeOfDay _fromTime = TimeOfDay.fromDateTime(DateTime.now());

  String get _title {
    switch (widget.type) {
      case PickerDemoType.date:
        return GalleryLocalizations.of(context).demoDatePickerTitle;
      case PickerDemoType.time:
        return GalleryLocalizations.of(context).demoTimePickerTitle;
    }
    return '';
  }

  String get _labelText {
    switch (widget.type) {
      case PickerDemoType.date:
        return DateFormat.yMMMd().format(_fromDate);
      case PickerDemoType.time:
        return _fromTime.format(context);
    }
    return '';
  }

  Future<void> _showDatePicker() async {
    final picked = await showDatePicker(
      context: context,
      initialDate: _fromDate,
      firstDate: DateTime(2015, 1),
      lastDate: DateTime(2100),
    );
    if (picked != null && picked != _fromDate) {
      setState(() {
        _fromDate = picked;
      });
    }
  }

  Future<void> _showTimePicker() async {
    final picked = await showTimePicker(
      context: context,
      initialTime: _fromTime,
    );
    if (picked != null && picked != _fromTime) {
      setState(() {
        _fromTime = picked;
      });
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        automaticallyImplyLeading: false,
        title: Text(_title),
      ),
      body: Center(
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            Text(_labelText),
            const SizedBox(height: 16),
            RaisedButton(
              child: Text(
                GalleryLocalizations.of(context).demoPickersShowPicker,
              ),
              onPressed: () {
                switch (widget.type) {
                  case PickerDemoType.date:
                    _showDatePicker();
                    break;
                  case PickerDemoType.time:
                    _showTimePicker();
                    break;
                }
              },
            )
          ],
        ),
      ),
    );
  }
}
```dart
