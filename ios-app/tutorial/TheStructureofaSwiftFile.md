一个 Swift 程序可以由一个文件或多个文件组成。在 Swift 中，一个文件是一个有意义的单元，并且对于可以进入其中的 Swift 代码的结构有一定的规则。只有Module，Variable declarations，Function declarations，Object type declarations可以放在 Swift 文件的顶层。

- Module import statements

  模块是比文件更高级别的单元。一个模块可以包含多个文件，这些文件都可以自动看到彼此。您的应用程序的文件属于一个模块并且可以相互看到。但是如果没有 import 语句，一个模块就看不到另一个模块。 import UIKit 就是在引用UIKit Module

- Variable declarations
  A variable declared at the top level of a file is a global variable: all code in any file will be able to see and access it, without explicitly sending a message to any object.

- Function declarations
  A function declared at the top level of a file is a global function: all code in any file will be able to see and call it, without explicitly sending a message to any object.

- Object type declarations
  The declaration for a class, a struct, or an enum.

This is a legal Swift file containing at its top level (just to demonstrate that it can be done) an import statement, a variable declaration, a function declaration, a class declaration, a struct declaration, and an enum declaration:

```swift
import UIKit
var one = 1
func changeOne() {
}
class Manny {
}
struct Moe {
}
enum Jack {
}
```

##### Example 1-1. Schematic structure of a legal Swift file

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
class Manny {
    let name = "manny"
    func sayName() {
        print(name)
    }
    class Klass {}
    struct Struct {}
    enum Enum {}
}
struct Moe {
    let name = "moe"
    func sayName() {
        print(name)
    }
    class Klass {}
    struct Struct {}
    enum Enum {}
}
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



# Namespaces

```
class Manny {
    class Klass {}
}
```

这种声明 Klass 的方式使 Klass 成为嵌套类型。它有效地将 Klass“隐藏”在 Manny 体内。 Manny 是一个命名空间！ Manny 内部的代码可以直接看到（和说）Klass。但是 Manny 之外的代码无法做到这一点。它必须显式指定名称空间才能通过名称空间所代表的障碍。为此，它必须首先说出 Manny 的名字，然后是一个点，然后是 Klass 一词。简而言之，它必须说Manny.Klass。

# Modules

当您导入一个模块时，该模块的所有顶级声明也对您的代码可见，而无需您显式使用模块的命名空间来引用它们。例如，NSString 所在的 Cocoa 的 Foundation 框架是一个模块。当你编写 iOS 程序时，你会说 import Foundation（或者，更可能的是，你会说 import UIKit，它本身会导入 Foundation），这样你就可以在不说 Foundation.NSString 的情况下谈论 NSString

Swift 本身是在一个模块中定义的——Swift 模块。但你不必导入它，因为你的代码总是隐式导入 Swift 模块。您可以通过使用 import Swift 行启动一个文件来明确这一点；没有必要这样做，但也没有坏处。

这个事实很重要，因为它解决了一个重大谜团：像 print 这样的东西从何而来，为什么可以在发送给任何对象的任何消息之外使用它们？ print 实际上是在 Swift 模块顶层声明的函数，你的代码可以看到 Swift 模块的顶层声明，因为它导入了 Swift。就您的代码而言，print 函数变成了与其他任何函数一样的普通顶级函数；它对您的代码是全局的，您的代码可以在不指定其名称空间的情况下使用它。你可以指定它的命名空间——像 Swift.print("hello") 这样的说法是完全合法的——但你可能永远不会这样做，因为你不需要这样做。



### 可执行代码不可以放在文件的顶层

