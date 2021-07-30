# Button宽度占满屏幕
在Button之外再套一层SizeBox,Container控制大小
```dart
SizedBox(
  width: double.infinity,
  height: 50,
  child: RaisedButton(
    onPressed: () {},
    child: Text("宽度占满了"),
    color: Colors.green,
    textColor: Colors.white,
  ),
),
```

# button disable
设置onPressed为null就是disable

```dart
RaisedButton(
  // shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(10.0)),
  // color: Color(0xFFF8bBD0),
  // icon: Icon(Icons.notifications),
  child: Padding(
    //上下左右各添加16像素补白
    padding: EdgeInsets.all(3.0),
    child: Text(label),
  ),
  onPressed:_isAgreeSpec? doPressed : null,
),
```