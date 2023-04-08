# 变量

类型名要以大写字母开头；变量名则以小写字母开头

```swift
let one = 1; // 只读
var two = 1；
one = two // compile error 
two = "hello" // 变量也是有类型的，其类型是在变量声明时创建的，而且永远不会改变
```



# 函数

Swift还有一个特殊的规则，那就是名为main.swift的文件可以在顶层包含可执行代码

```swift
func go() {
    let one = 1
    var two = 2
    two = one
}
go()
func sum (x:Int, _ y:Int) -> Int {
    let result = x + y
    return result
}
```



# Swift文件的结构

Swift程序可以包含一个或多个文件。在Swift中，文件是一个有意义的单元，它通过一些明确的规则来确定Swift代码的结构（假设不在main.swift文件中）。只有某些内容可以位于Swift文件的顶层部分，主要有如下几个部分

模块import语句

模块是比文件更高层的单元。一个模块可以包含多个文件，在Swift中，模块中的文件能够自动看到彼此；但如果没有import语句，那么一个模块是看不到其他模块的。比如，想想如何在iOS程序中使用Cocoa：文件的第1行会使用import UIKit。

变量声明

声明在文件顶层的变量叫作全局变量：只要程序还在运行，它就会一直存在。

函数声明

声明在文件顶层的函数叫作全局函数：所有代码都能看到并调用它，无需向任何对象发送消息。

```swift
import UIKit
var one = 1
func changeOne() {
    let two = 2
    func sayTwo() {
        print(two)
    }
    class Klass {}
    struct Struct {}
    enum Enum {}
    one = two
}
```



# 对象

类、结构体，枚举

- **可执行代码可以位于文件的顶部，因为这是不行的。只有函数体可以包含可执行代码**

```swift
import UIKit	// import语句
var one = 1		// 变量声明
func changeOne() {	// 函数声明
}
class Manny {	// 类声明
}
struct Moe {	// 结构体
}
enum Jack {		// 枚举
}

```

## 类

```swift
class Manny {
    let name = "manny"
    func sayName() {
        print(self.name)	// self 可选 默认self
    }
    class Klass {}
    struct Struct {}
    enum Enum {}
}

class Dog {
    var name = ""
    private var whatADogSays = "woof"
    func bark() {
        print(self.whatADogSays)
    }
    func speak() {
        print(self.whatADogSays)
    }
}

class Dog {
    var name = ""
    var license = 0
    init(name:String) {
        self.name = name
    }
    init(license:Int) {
        self.license = license
    }
    init(name:String, license:Int) {
        self.name = name
        self.license = license
    }
}

let dog1 = Dog () // 实例化类

```

### 类实例作为参数

```swift
class MyClass {
    var s = ""
    func store(s:String) {
            self.s = s
    }
}
let m = MyClass()
let f = MyClass.store(m) // what just happened!?
```

### 类继承

```swift
class Quadruped {
    func walk () {
        print("walk walk walk")
    }
}
class Dog : Quadruped {}
```

多态重载

```swift
class Dog {
    func bark() {
        print("woof")
    }
}
class NoisyDog : Dog {
    override func bark() {
        super.bark(); super.bark()
    }
}
```



## struct

```swift
struct Moe {
    let name = "moe"
    func sayName() {
        print(name)
    }
    class Klass {}
    struct Struct {}
    enum Enum {}
}

struct Greeting {
    static let friendly = "hello there"
    static let hostile = "go away"
}

```

## enum

```swift
enum Jack {
    var name : String {
        return "jack"
    }
    func sayName() {
        print(name)
    }
    class Klass {}
    struct Struct {}
    enum Enum {}
}
```

# 作用域与生命周期

在Swift程序中，一切事物都有作用域。这指的是它们会被其他事物看到的能力。一个事物可以嵌套在其他事物中，形成一个嵌套的层次结构。规则是一个事物可以看到与自己相同层次或是更高层次的事物。层次有：

·模块是一个作用域。

·文件是一个作用域。

·对象声明是一个作用域。

·花括号是一个作用域。

# 命名空间

命名空间指的是程序中的具名区域。命名空间具有这样一个属性：如果事先不穿越区域名这一屏障，那么命名空间外的事物是无法访问到命名空间内的事物的。这是一个好想法，因为通过命名空间，我们可以在不同地方使用相同的名字而不会出现冲突。显然，命名空间与作用域是紧密关联的两个概念。

```swift
class Manny {
    class Klass {}
}

Manny.Klass // 就可以调用
```



# 模块

顶层命名空间是模块。在默认情况下，应用本身就是个模块，因此也是个命名空间；大概来说，命名空间的名字就是应用的名字。比如，假如应用叫作MyApp，那么如果在文件顶层声明一个类Manny，那么该类的真实名字就是MyApp.Manny。但通常情况下是不需要这个真实名字的，因为代码已经在相同的命名空间中了，可以直接看到名字Manny。

框架也是模块，因此它们也是命名空间。比如，Cocoa的Foundation框架（NSString位于其中）就是个模块。在编写iOS程序时，你会import Foundation（还有可能import UIKit，它本身又会导入Foundation），这样就可以直接使用NSString而不必写成Foundation.NSString了。不过你可以写成Foundation.NSString，如果在自己的模块中声明了一个不同的NSString，那么为了区分它们，你就只能写成Foundation.NSString了。你还可以创建自己的框架，当然了，它们也都是模块。

代码总是会隐式导入Swift本身。还可以显式导入，方式就是以import Swift作为文件的开始；但没必要这么做事实上，print是在Swift.h头文件的顶层声明的一个函数——你的文件可以看到它，因为它导入了Swift。

# 查看头文件

你可以查看Swift.h文件并阅读和研究它，这么做很有帮助。要想做到这一点，请按住Command键并单击代码中的print。此外，还可以显式import Swift并按住Command键，然后单击Swift来查看其头文件！你看不到任何可执行的Swift代码，不过却可以查看到所有可用的Swift声明，包括如print之类的顶层函数、+之类的运算符以及内建类型的声明，如Int和String（查找struct Int、struct String等）。