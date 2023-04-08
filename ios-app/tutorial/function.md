```swift
func sum (_ x:Int, _ y:Int) -> Int {
    let result = x + y
    return result
}
```

# External Parameter Names

参数有内部名称和外部名称

默认参数内部名称和外部名称一样

```
func echo(string:String, times:Int) -> String {
    var result = ""
    for _ in 1...n { result += s }
    return result
}
echo(string: "hello", times: 5)
```

设置参数内部名称和外部名称

```
func echo(string s:String, times n:Int) -> String {
    var result = ""
    for _ in 1...n { result += s }
    return result
}

echo(string: "hello", times: 5)
```



不要参数外部名称

```
func echo(_ s:String, _ n:Int) -> String {
    var result = ""
    for _ in 1...n { result += s }
    return result
}

echo("hello", 5)
```

# Default Parameter Values

参数可以有默认值。这意味着调用者可以完全省略参数，不为其提供参数；该值将成为默认值

```
func echo(_ s:String, times:Int = 1) -> String {
    var result = ""
    for _ in 1...n { result += s }
    return result
}

echo("hello")
```

# Variadic Parameters

参数可以是可变的。这意味着调用者可以根据需要提供此参数类型的任意多个值，以逗号分隔；函数体将接收这些值作为数组。

```
func sayStrings(_ arrayOfStrings:String ...) {
    for s in arrayOfStrings { print(s) }
}
```

## 可修改的参数

```
func removeCharacter(_ c:Character, from s: inout String) -> Int {
    var howMany = 0
    while let ix = s.firstIndex(of:c) {
        s.remove(at:ix)
        howMany += 1
    }
    return howMany
}
```

## Reference Type Modifiable Parameters



# Overloading