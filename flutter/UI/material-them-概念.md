# 共通颜色
https://material.io/design/color/the-color-system.html#color-usage-and-palettes

## The baseline Material color theme

![](img\2020-11-04-14-20-53.png)

## Primary and secondary colors

![](img\2020-10-29-13-06-38.png)

**1. primary**
**2. primary variant**
**3. sencondary**


## Surface, background, and error colors

![](img\2020-10-29-13-10-31.png)

* **1. Background color** is found behind scrollable content
* **2. Surface colors** map to components such as cards, sheets, and menus
* **3. Error color** indicates errors in components, such as text fields

## “On” color naming

![](img\2020-10-29-13-22-46.png)

1. On Primary
2. On Secondary
3. On Surface
4. On Background
5. On Error

## 一个完整的dark them

![](img\2020-10-29-13-25-50.png)

1. Background (0dp elevation surface overlay)
2. Surface (with 1dp elevation surface overlay)
3. Primary
4. Secondary
5. On background
6. On Surface
7. On Primary
8. On Secondary**

# Material 颜色体系

## primarySwatch
结论：之所以定义了10种颜色是因为分了两种小于500是浅色调，大于500是用于深色调。

flutter的主题（build下面的theme）中有个主题颜色（primarySwatch）
primarySwatch中的颜色是调用 MaterialColor这种颜色类，内部会有一个主色，一个map存储固定了从50-900的10种主色周边的颜色。500是Primary color，200是Secondary color == accentColor。

![](img\2020-11-18-13-19-51.png)

1. Primary color
2. Secondary color
3. Light and dark variants

![](img\2020-11-18-13-30-59.png)

从colors.dart代码来看 500是主色调，
小于500是用于浅色调，也就是brightness == Brightness.light
大于500是用于深色调,也就是brightness == Brightness.dark

```dart
 const MaterialColor(int primary, Map<int, Color> swatch) : super(primary, swatch);

  /// The lightest shade.
  Color get shade50 => this[50]!;

  /// The second lightest shade.
  Color get shade100 => this[100]!;

  /// The third lightest shade.
  Color get shade200 => this[200]!;

  /// The fourth lightest shade.
  Color get shade300 => this[300]!;

  /// The fifth lightest shade.
  Color get shade400 => this[400]!;

  /// The default shade.
  Color get shade500 => this[500]!;

  /// The fourth darkest shade.
  Color get shade600 => this[600]!;

  /// The third darkest shade.
  Color get shade700 => this[700]!;

  /// The second darkest shade.
  Color get shade800 => this[800]!;

  /// The darkest shade.
  Color get shade900 => this[900]!;
}
```

primarySwach定义了10种颜色是怎么使用的呢
具体看theme_data.dart内でfactory ThemeData的定义就可以了，这里面具体定义了所有默认使用的颜色。
这里摘出一小部分

![](img\2020-11-18-14-24-46.png)


プロパティ | 説明
------|---
brightness | アプリの全体の明るさ。Brightness.lightと Brightness.darkがあります。
primaryColor | アプリの基本色になります。AppBarやTabBar、FloatingActionButtonなど、アプリのメインとなるWidgetの背景色がこれになります
primaryColorBrightness | primaryColorにのみ適用されるbrightnessです。下のprimaryColorLightとprimaryColorDarkのどちらを使うかが決まります。
primaryColorLight | Brightness.lightが指定されているときの primaryColor
primaryColorDark | Brightness.darkが指定されているときの primaryColor
canvasColor | MaterialType(Material Designの構成要素のこと、button``card``circleなどがある)のうち Material.canvasに適用される色です。
accentColor | アプリのアクセントカラーです。ScrollViewをいっぱいまでスクロールしたときにでるoverscroll edge effectなどの色になります
accentColorBrightness | accentColorにのみ適用されるbrightnessです。下のprimaryColorLightとprimaryColorDarkのどちらを使うかが決まります。
scaffoldBackgroundColor | ScaffoldWidgetの背景色です。
bottomAppBarColor | BottomAppBarWidgetの背景色です。
cardColor | MaterialWidgetにtype: MaterialType.cardを指定したときやCardWidgetの背景色です。
dividerColor | DividerWidgetの色です。
focusColor | Widgetがフォーカスされたときの色です。TextFieldの編集中の色などになります。
hoverColor | Widgetがホバーされたときの色です。
splashColor | InkWellのsplashの色です。inkwell ![](img\2020-11-18-15-24-55.png)
highlightColor | InkWellのhighlightの色です。
splashFactory | InkWellのsplashの効果をカスタマイズできるファクトリクラスを指定します。
selectedRowColor | 選択行の背景色です。デフォルトでは特に使われていません。
unselectedWidgetColor | 選択されていないWidgetの色です。CheckBox などの非選択時の色として、accentColorとの対で使われます。
disabledColor | 非有効化されたWidgetの色です。押せないCheckBoxの色などに使われます。
buttonTheme | RaisedButtonなどのButton系Widgetのデフォルトの設定に使われます。サイズ、色、内部のTextWidgetのスタイルなど、さまざまな値が指定できます。
toggleButtonsTheme | ToggleButtons版のButtonTheme。
buttonColor | RaisedButtonなどのButton系Widgetの背景色です。
secondaryHeaderColor | PaginatedDataTableのヘッダーの色です。
textSelectionColor | TextFieldなど選択可能なTextWidgetの選択時の色です。
cursorColor | TextFieldなどのカーソルの色です。
textSelectionHandleColor | TextFieldなどで、文字を選択しているときのカーソルの色です。
backgroundColor | 背景色のうち、primaryColorが使われない部分の色です。ProgressIndicatorの残りの部分の色などに使われます。
dialogBackgroundColor | Dialogの背景色です。
indicatorTheme | TabBarの選択時のインジケーターの色です。
hintColor | TextFieldのhintTextの色です。
errorColor | エラーの色です。TextFieldのエラー文の色などに使われます。
toggleableActiveColor | Switch``Radio``CheckBoxなどのトグルできるボタンの選択時の色です。
textTheme | TextWidgetの基本的なスタイルです。body1``body2``title``caption``buttonなどのタイプに対するTextStyleを詰め合わせたものです。
primaryTextTheme | primaryColorに対応するTextThemeです。
accentTextTheme | accentColorに対応するTextThemeです。
inputDecorationTheme | TextFieldなどに設定するInputDecorationのThemeです。hintや枠線などを指定できます。
iconTheme | IconWidgetのThemeです。サイズと色を指定できます。
primaryIconTheme | primaryColorに対応するIconThemeです。
accentIconTheme | accentColorにタイソウルIconThemeです。
sliderTheme | SliderWidgetのThemeです。サイズや形などを指定できます。
tabBarTheme | TabBarWidgetのThemeです。インジケーターのスタイルやサイズなどを指定できます。
tooltipTheme | TooltipWidgetのThemeです。サイズや形などを指定できます。
cardTheme | CardWidgetのThemeです。色や形を指定できます。
clipTheme | ClipCircleなどのClip系WidgetのThemeです。色や形などを指定できます。
platform | ThemeごとにTargetPlatformを指定するためのプロパティです。実際のTargetPlatformとは異なる場合もあります。
materialTapTargetSize | Material系のWidgetのタップ領域を指定できます。paddingとshrinkWrapから選べます。
applyElevationOverlayColor | マテリアルデザインでelevation(浮いているような影が出る)エフェクトを表現する時に、overlayColorを利用するかどうかのフラグです。ダークテーマでは影の表現が難しいため、影の表現がオフになります。
pageTransitionsTheme | 画面遷移にどのようなアニメーションを使うかというThemeです。AndroidとiOSそれぞれ別に指定できます。
appBarTheme | AppBarのThemeです。色やタイトルのスタイルを指定できます。
bottomAppBarTheme | BottomAppBarのThemeです。色や形を指定できます。
colorScheme | Widgetに制限されないカラーセットです。
snackBarTheme | SnackBarWidgetのThemeです。色や形を指定できます。
dialogTheme | DialogのThemeです。色やテキストのスタイルが指定できます。
floatingActionButtonTheme | FloatingActionButtonWidgetのThemeです。色や形が指定できます。
typography | どのロケールで、どのフォントを使うかを指定します。
cupertinoOverrideTheme | Cupertino系のWidget(iOSライクなデザインのWidget)のTheme
bottomSheetTheme | BottomSheetWidgetのThemeです。形などを指定できます。
popupMenuTheme | PopupMenuのThemeです。色やテキストのスタイルが指定できます。
bannerTheme | MaterialBannerWidgetのThemeです。
dividerTheme | DividerWidgetのThemeです。色やサイズなどを指定できます。
buttonBarTheme | ButtonBarWidgetのThemeです。サイズなどを指定できます。

# 具体颜色应用

## primarySwatch

* AppBarやbackdropなどの色
* BottomNavigationのselected(Brightness.lightの場合)
* TextFieldの選択した時の枠線やアイコンの色(InputDecorationの色)
* CircleAvatarの色
* DialogのButtonの色

![](img\2020-11-18-15-31-32.png)

## accentColor == secandary

* FloatingButtonの色
* checkbox/radioのselected(Brightness.lightの場合)
* indicatorの矢印の色
* DialogのButtonの文字色
* BottomNavigationのselected(Brightness.darkの場合)

![](img\2020-11-18-15-33-48.png)

## onSurface
* OutlineButtonの枠線の色。注意すべきは、この色に .withOpacity(0.12) をした色になるので薄い。
* disableボタンもこの色に合わせてくれる

![](img\2020-11-18-15-37-17.png) 


## buttonTheme

```dart
buttonTheme: ButtonThemeData(
textTheme: ButtonTextTheme.primary,
shape: RoundedRectangleBorder(
    /// Dialog以外のボタンの角に影響を与えることができる
    borderRadius: const BorderRadius.all(Radius.circular(24)),
    ),
),
```
![](img\2020-11-18-15-40-59.png)

## OutlineButtonの枠線の色をつけたい場合
ThemeDataだけではできません。直接指定する必要があります。

```dart
OutlineButton.icon(
    borderSide: BorderSide(
    color: Theme.of(context).primaryColor.withOpacity(0.5),
  ),
```

## iconTheme
* 普通のアイコンのテーマを変えられる。
* IconButton,Iconのどちらも変わる

```dart
iconTheme: const IconThemeData.fallback().copyWith(
    color: _accent,
  ),
```
![](img\2020-11-18-15-43-40.png)

## accentIconTheme

* FloatingActionButtonのアイコン色

```dart
 accentIconTheme: const IconThemeData.fallback().copyWith(
    color: Colors.red,
  ),
```
![](img\2020-11-18-15-44-47.png)

## FloatingActionButtonの色
ThemeDataだけでは変えられません。
直接指定する必要があります。ただし、 Theme.of(context)を使いましょう。

```dart
floatingActionButton: FloatingActionButton(
    backgroundColor: Theme.of(context).primaryColor,
```

## primaryIconTheme

* AppBarなどで利用しているアイコン
```dart
  primaryIconTheme: const IconThemeData.fallback().copyWith(
    color: Colors.greenAccent,
  ),
 ```

## primaryTextTheme
AppBarのアイコンを変えたら、こちらも変えたいでしょう。

* appBarのタイトルの色が変わる
```dart
  primaryTextTheme: const TextTheme().copyWith(
    title: TextStyle().copyWith(
      color: Colors.teal,
    ),
  ),
```

![](img\2020-11-18-15-47-37.png)


## フォーム
TextFieldなどの枠線の変更です。
動きがないとわからないかもしれないので、下の動画を見てください。

### inputDecorationTheme enabledBorder
* TextFieldなどで設定する枠線の設定を変えられる
利用しているWidgetで InputBorder を指定してもこちらの値が使われる

```dart  
  inputDecorationTheme: InputDecorationTheme(
    enabledBorder: UnderlineInputBorder(
      borderSide: BorderSide(color: _primary),
    ),
```

### inputDecorationTheme focusedBorder
* TextFieldなどでonFocusの時の枠線のstyleを設定
  ```dart  
    focusedBorder: OutlineInputBorder(
      borderSide: BorderSide(color: _accent),
    ),
  ```
### inputDecorationTheme labelStyle
* InputDecorationのlabelTextの色が変えられる
  ```dart  
    labelStyle: const TextStyle(        
      color: Colors.orange,
    ),
  ```

### labelStyle fillColor
* filed:trueの場所の色を設定できる。
  ```dart
    fillColor: Colors.orange.shade50,
  ```
  ![](img\2020-11-18-15-52-07.png)


# 排版字体

## Typography

![](img\2020-10-29-13-32-52.png)


# 获取ThemeData

* 直接获取
  `Theme.of(context).textTheme.body1`

* copyWith
  ```dart
    ThemeData.light().copyWith(
        accentColor: kShrineBrown900,
        primaryColor: kShrinePink100,
    ),
  ```

* merge
  ```dart
    Theme.of(context).textTheme.headline.merge(TextStyle(
        fontWeight: FontWeight.w700,
        fontSize: 35.0,
    )),
  ```

* apply
  ```dart
  style.apply(
    fontWeight: FontWeight.w700,
    fontSize: 35.0, 
  )
  ```
