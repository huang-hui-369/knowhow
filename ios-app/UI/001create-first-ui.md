https://developer.apple.com/cn/documentation/swiftui/



**SwiftUI**

file --> new --> project

![image-20220702201654544](.\001create-first-ui.assets\image-20220702201654544.png)

![image-20220702202423029](.\001create-first-ui.assets\image-20220702202423029.png)



```swift
import SwiftUI

struct ContentView: View {
    var body: some View {
        VStack {
            HStack {
                Image(systemName: "heart")
                    .foregroundColor(.red)
                // Spacer()
                Text("Hello world! 111")
                    .padding()
                    .foregroundColor(/*@START_MENU_TOKEN@*/.blue/*@END_MENU_TOKEN@*/)
                /*
                Text("Hello2")
                 .padding()
                */
            }.offset(x: 0, y: 50.0)
            Spacer()
            Slider(value: Binding.constant(1))
        }
        // .padding(.horizontal, 50)
        .frame(width: UIScreen.main.bounds.size.width * 0.7)
    }
}
```

