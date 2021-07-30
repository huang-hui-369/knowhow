### 空类型声明符 ?
```dart
//dart 声明可空类型
class Student {
  String? name;//静态检查通过，使用 ? 表示可空类型，String? 则是可空类型 String
  int age; //静态检查错误
}
```

### 非空断言 !
```dart
//dart 非空断言
void main() {
  String? name;
  if (name!=null) {//其实已经做了判空处理，运行实际上是 OK 的，
    // 但是编译器无法智能推断是否进行判空，所以还是提示报错
    print(name!.length);
  } else {
    print(0);
  }

```

### required 关键字
@required 作为内置修饰符。主要用于允许根据需要标记任何命名参数（函数或类），使得它们不为空。

```dart
void testFunc({required String param}) {//required 修饰符
    //do logic
}
```

## ?.
使用 ?. 来代替 . ， 可以避免因为左边对象可能为 null ， 导致的异常：
// 如果 p 为 non-null，设置它变量 y 的值为 4。
p?.y = 4;

## ??

InputDecoration effectiveDecoration = (decoration ?? const InputDecoration())