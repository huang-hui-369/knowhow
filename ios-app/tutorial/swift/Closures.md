## Closures

拿闭包和函数比较的话，就容易理解闭包。函数是闭包的一种特例。您可以将函数视为命名闭包。闭包与函数的不同之处在于它们具有更紧凑和轻量级的语法。它们允许您编写一个“类函数”结构，而不必给它一个名称和一个完整的函数声明。其实感觉闭包经常用于call Back函数

### 定义闭包

`in` keyword used to separate the closure signature from the closure implementation.

```swift
let compareAscending = { (i: Int, j: Int) -> Bool in
    return i < j
}

compareAscending(42, 2)                                      false
compareAscending(-2, 12)                                     true
```

### 使用闭包

numbers.sort函数需要一个闭包作为by参数（call Back函数）

```swift
let compareAscending = { (i: Int, j: Int) -> Bool in
    return i < j
}

var numbers = [42, 9, 12, -17]                               [42, 9, 12, -17]
numbers.sort(by: compareAscending)                           [-17, 9, 12, 42]
```

