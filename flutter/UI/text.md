### 改行する

```dart
Text(
  '2行に渡る文字列だよ。\n2行に渡る文字列だよ。',
)
```
のように \n を入れます。他の言語と似てます。引用符が " でも ' でも改行の仕方は一緒です。

### 装飾を施す
装飾を施す場合は、 TextStyle を使います。 Flutter には予め、 Colors というクラスにいくつもの色が定義済みになっています。 今回は、 Colors.red を利用してみましょう。
```dart
Text(
  '赤い文字列だよ。',
  style: TextStyle(
    color: Colors.red,
  ),
)
```

### 行間を空ける
今のところ、太字にしたい場合、

```dart
Text(
  '2行に渡る文字列だよ。\n2行に渡る文字列だよ。',
  style: TextStyle(
    height: 1.0,
  ),
)
```

### 太字にする

Text(
  '太い文字列だよ。ABC',
  style: TextStyle(
    fontWeight: FontWeight.bold,
  ),
)
のようにします。

現在の仕様では、太字は英数字のみに対応のようです。

### 変数の内容や値を文字列の中に表示する
$ という記号を使うことで、文字列の中に、変数の内容を挿入することができます。

```dart
final someVariable = "へんすう";
Text(
  '変数 $someVariable を表示する方法。',
)
```

直接、値を入れたいときや、変数と文字列の境がわからないときには、 ${} を使います。

```dart
final someVariable = "Some Variable";
Text(
  'Show ${1} or ${someVariable}s.',
)
```

### 末尾を … で切る
引数 overflow に TextOverflow.ellipsis を適応することで、表示を強制的に一行に収めることができます。

```dart
Text(
  'とても長い文字列です。とても長い文字列です。とても長い文字列です。とても長い文字列です。とても長い文字列です。',
  overflow: TextOverflow.ellipsis,
),
```