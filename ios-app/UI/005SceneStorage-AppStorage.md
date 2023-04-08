## AppStorage

是一个全局的存储，它是使用UserDefaults来做持久化的，所以我们可以在app中任何地方获取使用它。它也是用于轻量级存储，例如app的设置信息。

AppStorage是需要声明一个唯一的key来代表要存的数据。它像其他的属性绑定器一样，可以获取它的binding来使用。

- 数据可持久化，app 退出后数据仍保留
- 仅包装了 UserDefault，数据可以 UserDefault 正常读取
- 可保存的数据类型同 UserDefault，不适合保存复杂类型数据

```swift
struct ContentView: View {
    @AppStorage("username") var username: String = "Anonymous"
    var body: some View {
        VStack {
            Text("Welcome, \(username)!")
            Button("Log in") {
                username = "@twostraws"
            }
        }
    }
}
```

上面的代码中使用了一个**`SwiftUI`**中的**`@AppStorage`**属性包装器，利用**`Button`**来修改属性**`userName`**的值，同时**`@AppStorage`**附加在属性上的逻辑会将**`"@twostraws"`**这个**`value`**通过**`key`**为**`"username"`**保存到**`UserDefaults`**中，并监听这个**`value`**值的改变，当**`value`**值改变时，**`SwiftUI`**会通知所有持有这个属性值的**`view`**进行刷新，故当点击**`Button`**时，**`Text`**的显示内容会从**`"Welcome, Anonymous !"`**变为**`"Welcome, @twostraws !"`**。

## UserDefaults

默认情况下使用的是UserDefaults.standard，也可以指定其他的UserDefaults。

```swift
public extension UserDefaults {
    static let shared = UserDefaults(suiteName: "group.com.fatbobman.examples")!
}

@AppStorage("userName",store:UserDefaults.shared) var name = "fat"
```

对UserDefaults操作将直接影响对应的@AppStorage

```swift
UserDefaults.standard.set("bob",forKey:"username")
```

UserDefaults是一种高效且轻量的持久化方案，它有以下不足：

- 数据不安全

  它的数据相对容易提取，所以不要保存和隐私有关的重要数据

- 持久化时机不确定

  为了效率的考量，UserDefaults中的数据在发生变化时并不会立即持久化，系统会在认为合适的时机才将数据保存在硬盘中。因此，可能发生数据不能完全同步的情况，严重时有数据彻底丢失的可能。尽量不要在其中保存会影响App执行完整性的关键数据，在出现数据丢失的状况下，App仍可根据默认值正常运行

尽管@AppStorage是作为UserDefaults的属性包装器存在的，但@AppStorage并没有支持全部的`property list`数据类型，目前仅支持：Bool、Int、Double、String、URL、Data（UserDefaults支持更多的类型）。

### 增加@AppStorage支持的数据类型

除了上述的类型外，@AppStorage还支持符合`RawRepresentable`协议且`RawValue`为`Int`或`String`的数据类型。通过增加`RawRepresentable`协议的支持，我们可以在@AppStorage中读取存储原本并不支持的数据类型。

下面的代码添加了对`Date`类型的支持：

```swift
extension Date:RawRepresentable{
    public typealias RawValue = String
    public init?(rawValue: RawValue) {
        guard let data = rawValue.data(using: .utf8),
              let date = try? JSONDecoder().decode(Date.self, from: data) else {
            return nil
        }
        self = date
    }

    public var rawValue: RawValue{
        guard let data = try? JSONEncoder().encode(self),
              let result = String(data:data,encoding: .utf8) else {
            return ""
        }
       return result
    }
}
```

使用起来和直接支持的类型完全一致：

```swift
@AppStorage("date") var date = Date()
```

下面的代码添加了对`Array`的支持：

```swift
extension Array: RawRepresentable where Element: Codable {
    public init?(rawValue: String) {
        guard let data = rawValue.data(using: .utf8),
              let result = try? JSONDecoder().decode([Element].self, from: data)
        else { return nil }
        self = result
    }

    public var rawValue: String {
        guard let data = try? JSONEncoder().encode(self),
              let result = String(data: data, encoding: .utf8)
        else {
            return "[]"
        }
        return result
    }
}
@AppStorage("selections") var selections = [3,4,5]
```



### 安全和便捷的声明（一）

@AppStorage的声明方式有两个令人不悦的地方：

- 每次都要设定Key（字符串）
- 每次都要设定默认值

而且开发者很难享受到代码自动补全和编译时检查带来的快捷、安全的体验。

较好的解决方案是将@AppStorage集中声明，并在每个视图中通过引用注入。**鉴于SwiftUI的刷新机制，我们必须要在集中声明、单独注入后仍需保留@AppStorage的`DynamicProperty`特征——当UserDefaults的值发生变动时刷新视图。**

下面的代码能满足以上的要求：

```swift
enum Configuration{
    static let name = AppStorage(wrappedValue: "fatbobman", "name")
    static let age = AppStorage(wrappedValue: 12, "age")
}
```

在视图中使用方法如下：

```swift
let name = Configuration.name
var body:some View{
     Text(name.wrappedValue)
     TextField("name",text:name.projectedValue)
}
```

`name`和直接在代码中通过@AppStorage声明的效果类似。不过付出的代价就是需要将`wrappedValue`和`projectedValue`明确标注出来。

> 是否有不标注`wrappedValue`和`projectedValue`又能达到上述结果的实现方案呢？在**安全和便捷的声明（二）**中我们将尝试使用另一种解决途径。

### 集中注入

在介绍另一种便捷声明方式之前，我们先聊一下集中注入的问题。

【健康笔记3】目前面临着前言中所描述的情况，配置信息内容很多，如果单独注入会很麻烦。我需要找到一种可以集中声明、一并注入的方式。

在**安全和便捷的声明（一）**中使用的方法对于单独注入的情况是满足的，但如果我们想统一注入的话就需要其他的手段了。

> 我并不打算将配置数据汇总到一个结构体中并通过支持`RawRepresentable`协议统一保存。除了数据转换导致的性能损失外，另一个重要问题是，如果出现数据丢失的情况，逐条保存的方式还是可以保护绝大多数的用户设定的。

在**基础指南**中，我们提到@AppStorage在视图中的表现同@State非常类似；不仅如此，@AppStorage还有一个官方文档从没提到的神奇特质，**在ObservableObject中具有同@Published一样的特性——其值发生变化时会触发`objectWillChange`**。这个特性只发生在@AppStorage身上，@State、@SceneStorage都不具备这个能力。

*目前我无法从文档或暴露的代码中找到这一特性原因，因此以下的代码并不能获得官方的长期保证*

```swift
class Defaults: ObservableObject {
    @AppStorage("name") public var name = "fatbobman"
    @AppStorage("age") public var age = 12
}
```

视图代码：

```swift
@StateObject var defaults = Defaults()
...
Text(defaults.name)
TextField("name",text:defaults.$name)
```

不仅代码整洁了许多，而且由于只需要在`Defaults`中声明一次，极大的降低了由于字符串拼写错误而出现的不易排查的Bug。

> `Defaults`中使用的是`@AppStorage`的声明方式，而`Configuration`中使用的是`AppStorage`的原始构造形式。变化的目的是为了能够保证视图更新机制的正常运作。

### 安全和便捷的声明（二）

**集中注入**中提供的方法已经基本解决了我在当前使用@AppStorage中碰到的不便,不过我们还可以尝试另一种优雅、有趣的逐条声明注入的方式。

首先修改一下`Defaults`的代码

```swift
public class Defaults: ObservableObject {
    @AppStorage("name") public var name = "fatbobman"
    @AppStorage("age") public var age = 12
    public static let shared = Defaults()
}
```

创建一个新的属性包装器`Default`

```swift
@propertyWrapper
public struct Default<T>: DynamicProperty {
    @ObservedObject private var defaults: Defaults
    private let keyPath: ReferenceWritableKeyPath<Defaults, T>
    public init(_ keyPath: ReferenceWritableKeyPath<Defaults, T>, defaults: Defaults = .shared) {
        self.keyPath = keyPath
        self.defaults = defaults
    }

    public var wrappedValue: T {
        get { defaults[keyPath: keyPath] }
        nonmutating set { defaults[keyPath: keyPath] = newValue }
    }

    public var projectedValue: Binding<T> {
        Binding(
            get: { defaults[keyPath: keyPath] },
            set: { value in
                defaults[keyPath: keyPath] = value
            }
        )
    }
}
```

现在我们可以在视图中采用如下代码来逐个声明注入了：

```swift
@Default(\.name) var name
Text(name)
TextField("name",text:$name)
```

逐个注入且无需标注`wrappedValue`和`projectedValue`。由于使用`keyPath`，避免了可能出现的字符串拼写错误问题。

鱼和熊掌不可兼得，上述的方法还是不十分完美——会出现过度依赖的情况。即使你只在视图中注入了一个UserDefaults键值（比如`name`），但当`Defaults`中其他未注入的键值内容发生变动时（`age`发生变化），依赖`name`的视图也同样会被刷新。

不过由于通常情况下配置数据的变化频率很低，所以并不会对App造成什么性能负担。



## SceneStorage

是一个属性绑定器，它可以存在于每一个scene中。它只在Views中能被获取到

根据下图例子，我们使用SceneStorage需要声明一个唯一的key来代表要存的数据，然后我们可以像使用State一样来使用这个对象，SwiftUI会自动帮我们存储和恢复这个对象。

使用方法同@AppStorage 十分类似，不过其作用域仅限于当前 Scene。

- 数据作用域仅限于 Scene 中
- 生命周期同 Scene 一致，当前在 PadOS 下，如果强制退出一个两分屏显示的 app, 系统在下次打开 app 时有时会保留上次的 Scene 信息。不过，如果如果单独退出一个 Scene，数据则失效
- 支持的类型基本等同于@AppStorage，适合保存轻量数据
- 比较适合保存基于 Scene 的特质信息，比如 TabView 的选择，独立布局等数据



参考

https://www.jianshu.com/p/9bd5f7510fe6



