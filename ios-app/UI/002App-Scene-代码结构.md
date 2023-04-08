##### 基本概念

------

**App**

SwiftUI2.0 提供的全新协议。通过声明一个符合 App 协议的结构来创建一个程序，并通过计算属性 body 来实现程序的内容。

- 通过@main(swift5.3 新特性）设定程序的入口，每个项目只能有一个进入点
- 管理整个 app 的生命周期
- 在这个作用域下声明的常量、变量其生命周期与整个 app 是完全一致的。

**Scene**

场景是视图（View）层次结构的容器。通过在 App 实例的 body 中组合一个或多个符合 Scene 协议的实例来呈现具体程序。

- 生命周期由系统管理
- 系统会根据运行平台的不同而调整场景的展示行为（比如相同的代码在 iOS 和 macOS 下的呈现不同，或者某些场景仅能运行于特定的平台）
- SwiftUI2.0 提供了几个预置的场景，用户也可以自己编写符合 Scene 协议的场景。上述代码中便是使用的一个预置场景 WindowGroup

通过 App 和 Scene 的加入，绝不是仅仅减少代码量这么简单。通过这个明确的层级设定，我们可以更好的掌握在不同作用域下各个部分的生命周期、更精准数据传递、以及更便利的多平台代码共享。本文后面会用具体代码来逐个阐述。

*App 和 Scene 都是通过各自的 functionBuilder 来解析的，也就是说，新的模板从程序的入口开始便是使用 DSL 来描述的。*

##### 程序系统事件响应

------

由于去除了 AppDelegate.swift 和 SceneDelegate.swift，SwiftUI2.0 提供了新的方法来让程序响应系统事件。

**针对 AppDelegate.swift**

在 iOS 系统下，通过使用@UIApplicationDelegateAdaptor 可以方便的实现之前 AppDelegate.swfit 中提供的功能：

@main

struct NewAllApp: App {

   @UIApplicationDelegateAdaptor(AppDelegate.self) var appDelegate

​    var body: some Scene {

​        WindowGroup {

​            Text("Hello world")

​        }

​    }

}

**class** AppDelegate:NSObject,UIApplicationDelegate{

​    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey : Any]? = **nil**) -> Bool {

​        print("launch")

​        **return** **true**

​    }

}

由于目前还是测试版，虽然很多的事件已经定义，但现在并没有响应。估计很快会增加修改过来

**针对 SceneDelegate.swift**

通过新增添的 EnvironmentKey **scenePhase** 和新的**.onChange** 方法，SwiftUI 提供了一个更加有趣的场景事件解决方案：

@main

struct NewAllApp: App {

​    @Environment(\.scenePhase) var phase

​    var body: some Scene {

​        WindowGroup {

​           ContentView()

​        }

​        .onChange(**of**: phase){phase **in**

​                switch phase{

​                case .active:

​                    print("active")

​                case .inactive:

​                    print("inactive")

​                case .background:

​                    print("background")

​                @unknown default:

​                    print("for future")

​                }

​          }

​    }

}

同样是由于测试版的原因，该响应目前并没有完成。不过这段代码目前来看是 iOS 和 macOS 都通用的

**更新**

目前发现如果在 View 中，可以获取 scenePhase 的状态更新。下来代码目前可以正常执行

struct ContentView:View{ 

  @Environment(\.scenePhase) private var scenePhase

 var body: some Scene {

  WindowGroup {

   ContentView()

  }

  .onChange(**of**: phase){phase **in**

​                switch phase{

​                case .active:

​                    print("active")

​                case .inactive:

​                    print("inactive")

​                case .background:

​                    print("background")

​                @unknown default:

​                    print("for future")

​                }

​         }

   }

}

##### 预置场景

------

- **WKNotificationScene** 仅适用于 watchOS7.0，用于响应指定类别的远程或本地通知。目前还没有研究。
- **WindowGroup**

最常用的场景，可以呈现一组结构相同的窗口。使用该场景，我们无需在代码上做修改，只需要在项目中设定是否支持多窗口，系统将会按照运行平台的特性自动管理。

在 iOS 中，只能呈现一个运行窗口。

在 PadOS 中（如打开多窗口支持），最多可以打开两个运行窗口，可以分屏显示，也可以全屏独立显示。

在 macOS 中，可以打开多个窗口，并通过程序菜单中的窗口菜单来进行多窗口管理。

最开始的代码在三个平台下的状态：

![img](http://123.57.164.21/wp-content/uploads/2021/12/image-256-1024x1019.png)

如果在一个 WindowGroup 里加入多个 View, 呈现状态有点类似 VStack。

在一个 Scene 中加入多个 WindowGroup，只有最前面的可以被显示。

- **DocumentGroup**

创建一个可处理指定文件类型的窗口。在 iOS 和 PadOS 下都首先会呈现文件管理器，点击文件，进入对应的 View 来处理。macOS 下，通过菜单中的文件操作来选择或创建文件。

通过创建一个符合 FileDocument 的结构来定义支持哪种格式，以及打开和保存的工作。

//纯文本格式文件。write 的方法用于描述如何写入文件，如果不需写入可为空。

  struct TextFile: FileDocument {

​      static var readableContentTypes = [UTType.plainText]

​      var text = ""

​      init(initialText: String = "") {

​          text = initialText

​      }

​      init(fileWrapper: FileWrapper, contentType: UTType) throws {

​          **if** let data = fileWrapper.regularFileContents {

​              text = String(decoding: data, as: UTF8.self)

​          }

​      }

​      func write(to fileWrapper: inout FileWrapper, contentType: UTType) throws {

​          let data = Data(text.utf8)

​          let file = FileWrapper(regularFileWithContents: data)

​          fileWrapper = file

​      }

  }

  //图片文件，由于需要转换成 UIImage，该代码只支持 iOS 或 PadOS

  \#if os(iOS)

  struct ImageFile: FileDocument {

​      static var readableContentTypes = [UTType.image]

​      var image = UIImage()

​      init(initialImage: UIImage = UIImage()) {

​          image = initialImage

​      }

​    

​      init(fileWrapper: FileWrapper, contentType: UTType) throws {

​          **if** let data = fileWrapper.regularFileContents {

​              image =   UIImage(data: data) ?? UIImage()

​          }

​      }

  

​      func write(to fileWrapper: inout FileWrapper, contentType: UTType) throws { }

  }

  \#endif

调用

  import SwiftUI

  \#if os(iOS)

  import UIKit

  \#endif

  import UniformTypeIdentifiers

  

  @main

  struct NewAllApp: App {

​     @SceneBuilder var body: some Scene {

​          //可读写

​          DocumentGroup(newDocument: TextFile()) { file **in**

​                  TextEditorView(document: file.$document)

​          }

​          

​          \#if os(iOS)

​          //只读

​          DocumentGroup(viewing: ImageFile.self) { file **in**

​                  ImageViewerView(file: file.$document)

​            }

​          \#endif

​      }

  }

  

  struct TextEditorView: View {

​      @Binding var document: TextFile

​      @State var name = ""

​      var body: some View {

​          VStack{

​          TextEditor(text: $document.text)

​              .padding()

​          }

​          .background(Color.gray)

​      }

  }

  

  \#if os(iOS)

  struct ImageViewerView:View{

​      @Binding var document:ImageFile

​      var body: some View{

​          Image(uiImage: document.image)

​              .resizable(resizingMode: .stretch)

​              .aspectRatio(contentMode: .fit)

​      }

  }

  \#endif

![img](http://123.57.164.21/wp-content/uploads/2021/12/image-257-1022x1024.png)

可以将多个 DocumentGroup 放入 Scene 中，程序将会一并支持每个 DocumentGroup 所定义的文件类型。上述代码使程序可以创建、编辑纯文本文件，并且可以浏览图片文件。

在 macOS 上，需要在 macOS.entitlements 中设置 com.apple.security.files.user-selected.read-write 为真才能完成写入。

当在 Scene 中加入多个场景时，需要使用@SceneBuilder 或用 Group 将多个场景涵盖起来。

macOS 下当同时加入 WindowGroup 和 DocumentGroup 时，两个功能都可以正常运行。iOS 或 PadOS 下，只有顺序在最前面的被显示。

由于测试版的原因，目前仍有大量的功能无法实现或有问题。比如仍无法在 iOS 上通过 fileDocument 提供的 filename 来设置文件名，或者无法在创建新文件时选择格式等

- **Settings**

只用于 macOS, 用于编写程序的偏好设置窗口。

\#if os(macOS)

​      Settings{

​        Text("偏好设置").padding(.all, 50)

​      }

  \#endif

![img](http://123.57.164.21/wp-content/uploads/2021/12/image-258-1024x574.png)

##### 其他

------

- **onChange** 监视指定的值，在值改变时执行指定的 action。在 scenePhase 的用法介绍中有使用的范例
- **onCommands** 在 macOS 下设置程序的菜单。具体的使用方法请查看 [SwiftUI2.0 —— Commands（macOS 菜单）](https://www.fatbobman.com/posts/swiftUI2-commands/)
- **defaultAppStorage** 如果不想使用系统缺省 UserDefault.standard，可以自行设置存储位置，使用的几率不高。

##### 小结

------

至此，本文简单介绍了 SwiftUI2.0 新增的 App 和 Scene，下篇文章我们将探讨在新的层次结构下如何组织我们代码的 Data Flow。

当前的@AppBuilder 和@SceneBuilder 的功能都十分的基础，不包含任何的逻辑判断功能，因此目前我还没有办法实现根据条件来选择性的展示所需的 Scene。相信苹果应该会在未来增加这样的能力

*本文的代码为了能够在多平台使用，所以增加了不少编译判断，如果你只是在 iOS, 或 macOS 下开发 SwiftUI，则可根据各自平台简化代码。另外 Xcode12 中的代码补全对于 Target 的设定很敏感，如果你发现无法对某些平台的特定语句进行补全，请查看是否将 Scheme 设置到对应的平台。*

**@AppStorage**

AppStorage 是苹果官方提供的用于操作 UserDefault 的属性包装器。这个功能在 Swift 提供了 propertyWrapper 特性后，已经有众多的开发者编写了类似的代码。功能上没有任何特别之处，不过名称对应了新的 App 协议，让人更容易了解其可适用的周期。

- 数据可持久化，app 退出后数据仍保留
- 仅包装了 UserDefault，数据可以 UserDefault 正常读取
- 可保存的数据类型同 UserDefault，不适合保存复杂类型数据
- 在 app 的任意 View 层级都可适用，不过在 app 层使用并不起作用（不报错）

@main

struct AppStorageTest: App {

​    //不报错，不过不起作用

​    //@AppStorage("count") var count = 0

​    var body: some Scene {

​        WindowGroup {

​            RootView()

​            CountView()

​        }

​    }

}

struct RootView: View {

​    @AppStorage("count") var count = 0

​    var body: some View {

​        List{

​            Button("+1"){

​                count += 1

​            }

​        }

​    }

}

struct CountView:View{

​    @AppStorage("count") var count = 0

​    var body: some View{

​        Text("Count:\(count)")

​    }

}

**@SceneStorage**

使用方法同@AppStorage 十分类似，不过其作用域仅限于当前 Scene。

- 数据作用域仅限于 Scene 中
- 生命周期同 Scene 一致，当前在 PadOS 下，如果强制退出一个两分屏显示的 app, 系统在下次打开 app 时有时会保留上次的 Scene 信息。不过，如果如果单独退出一个 Scene，数据则失效
- 支持的类型基本等同于@AppStorage，适合保存轻量数据
- 比较适合保存基于 Scene 的特质信息，比如 TabView 的选择，独立布局等数据

@main

struct NewAllApp: App {

​    var body: some Scene {

​        WindowGroup{

​            ContentView1()

​        }

​    }

}

struct ContentView:View{

​    @SceneStorage("tabSeleted") var selection = 2

​    var body:some View{

​        TabView(selection:$selection){

​            Text("1").tabItem { Text("1") }.tag(1)

​            Text("2").tabItem { Text("2") }.tag(2)

​            Text("3").tabItem { Text("3") }.tag(3)

​        }

​    }

}

![img](http://123.57.164.21/wp-content/uploads/2021/12/image-259-1024x631.png)

上述代码在 PadOS 下运行正常，不过在 macOS 下程序会报错。估计应该是 bug

##### Data Flow

------

- **手段**

苹果在 SwiftUI2.0 中添加了@AppStorage @SceneStorage @StateObject 等新的属性包装器，我根据自己的理解对目前 SwiftUI 提供的部分属性包装器做了如下总结：

![img](http://123.57.164.21/wp-content/uploads/2021/12/image-260-1024x574.png)

经过此次升级后，SwiftUI 已经大大的完善了各个层级数据的生命周期管理，对不同的类型、不同的场合、不同的用途都提供了解决方案，为编写符合 SwiftUI 的 Data Flow 提供了便利，我们可以根据自己的需要选择适合的 Source of truth 手段。

- **变化**

在 SwiftUI1.0 中，我们通常会在 AppDelegate 中创建需要生命周期与 app 一致的数据（比如 CoreData 的 Container），在 SceneDelegate 中创建 Store 之类的数据源，并通过。environmentObject 注入。不过随着 SwiftUI2.0 在程序入口方面的变化，以及采取的全新 Delegate 响应方式，我们可以通过更简洁、清晰的代码完成上述工作。

@main

struct NewAllApp: App {

​    @StateObject var store = Store()

​    var body: some Scene {

​        WindowGroup{

​            ContentView()

​                .environmentObject(store)

​        }

​    }

}

**class** Store:ObservableObject{

​    @Published var count = 0

}

上述例子中，将

@StateObject var store = Store()

换成

let store = Store()

目前来说是一样的。

*虽然目前 SceneBuilder、CommandBuilder 对 Dynamic update 和逻辑判断尚不支持，我相信应该在不久的将来，或许我们就可以使用类似下面的代码来完成很多有趣的工作了，**当前代码无法执行***

@main

struct NewAllApp: App {

​    @StateObject var store = Store()

​    @SceneBuilder var body: some Scene {

​        //@SceneBuilder 目前不支持判断，不过将来应该会加上

​        **if** store.scene == 0 {

​        WindowGroup{

​            ContentView1()

​                .environmentObject(store)

​        }

​        .onChange(**of**: store.number){ value **in**

​            print(value)

​        }

​        .commands{

​            CommandMenu("DynamicButton"){

​                //目前无法动态切换内容，怀疑是 bug，已反馈

​                switch store.number{

​                case 0:

​                    Button("0"){}

​                case 1:

​                    Button("1"){}

​                default:

​                    Button("other"){}

​                }

​            }

​        }

​        **else** {

​         DocumentGroup(newDocment:TextFile()){ file **in**

​              TextEditorView(document:file.$document)

​         }

​        }

​        

​        Settings{

​            VStack{

​               //可正常变换

​                Text("\(store.number)")

​                    .padding(.all, 50)

​            }

​        }

​    }

}

struct ContentView1:View{

​    @EnvironmentObject var store:Store

​    var body:some View{

​        VStack{

​        Picker("select",selection:$store.number){

​            Text("0").tag(0)

​            Text("1").tag(1)

​            Text("2").tag(2)

​        }

​        .pickerStyle(SegmentedPickerStyle())

​        .padding()

​        }

​    }

}

**class** Store:ObservableObject{

​    @Published var number = 0

​    @Published var scene = 0

}

- **跨平台代码**

在 [上篇文章](https://www.fatbobman.com/posts/swiftui2-new-feature-1/) 我们介绍了新的@UIApplicationDelegateAdaptor 的使用方法，我们也可以直接创建一个支持 Delegate 的 store。

import SwiftUI

**class** Store:NSObject,ObservableObject{

​    @Published var count = 0

}

\#if os(iOS)

extension Store:UIApplicationDelegate{

​    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey : Any]? = **nil**) -> Bool {

​        print("launch")

​        **return** **true**

​    }

}

\#endif

@main

struct AllInOneApp: App {

​    \#if os(iOS)

​    @UIApplicationDelegateAdaptor(Store.self) var store

​    \#else

​    @StateObject var store = Store()

​    \#endif

​    

​    @Environment(\.scenePhase) var phase

​    @SceneBuilder var body: some Scene {

​            WindowGroup {

​                RootView()

​                    .environmentObject(store)

​            }

​            .onChange(**of**: phase){phase **in**

​                switch phase{

​                case .active:

​                    print("active")

​                case .inactive:

​                    print("inactive")

​                case .background:

​                    print("background")

​                @unknown default:

​                    print("for future")

​                }

​            }

​      

​        \#if os(macOS)

​        Settings{

​            Text("偏好设置").padding(.all, 50)

​        }

​        \#endif

​    }

}

##### 总结

------

在 [ObservableObject 研究——想说爱你不容易](https://www.fatbobman.com/posts/observableObject-study/) 中，我们探讨过 SwiftUI 更倾向于我们不要创建一个沉重的 Singel source of truth, 而是将每个功能模块作为独立的状态机（一起组合成一个大的状态 app），使用能够对生命周期和作用域更精确可控的手段创建区域性的 source of truth。

从 SwiftUI 第一个版本升级的内容来看，目前 SwiftUI 仍是这样的思路。

