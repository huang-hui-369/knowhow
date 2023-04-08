# 從 storyboard 畫面切換到 SwiftUI 畫面

我們也可以利用 Container View 將 SwiftUI view 變成 view controller 裡某個區塊的畫面。

假設我們有兩個畫面，如下圖所示，

第一個畫面是 storyboard 裡的 view controller，我們希望點選 controller 上的 button 後切換到 第二个显示详细歌词的SwiftUI 的 view 

1. storyboard的第一个画面

   ![image-20220703172351366](.\099storyboard中嵌入swiftUI.assets\image-20220703172351366.png)

2. 第二个显示详细歌词的SwiftUI 的 view

   ![image-20220703172517868](.\099storyboard中嵌入swiftUI.assets\image-20220703172517868.png)

   ```swift
   import SwiftUI
   struct LyricsView: View {
       var body: some View {
           Text("""
   嘿 有些話我沒說清楚 是故意讓它變模糊
   也為了避免 過於理智的衝突
   嘿 有些事我沒有宣布 是無疑讓它更懸殊
   也為了減少 重覆彼此不舒服
   因為太有感觸 所以才會如此反撲
   任一時糊塗 違規的信徒 恍恍惚惚
   誰當眾地退出 主動地刪除 都視若無睹
   你才真的可惡 莫名傷得體無完膚
   為認真演出 結束得倉促 吞吞吐吐
   誰抱頭地痛哭 熱情地歡呼 都可有可無
   """)
           .padding()
               .background(Color.yellow)
       }
   }
   ```

   

3. 设置画面迁移

   當我們想從 storyboard 拉 segue 切換頁面時，每個畫面都要有個控制它的 controller ，因此我們只要能將 SwiftUI 的 view 包在一個 controller 裡，即可順利地切換到 SwiftUI 的畫面。

   1. 從 Storyboard 加入 UIHostingController

      如下圖所示，我們可從 storyboard 的 Objects library 視窗裡找到 Hosting View Controller，直接將它加到 storyboard 設計畫面的畫布上。

      UIHostingController 顯示的畫面將是 SwiftUI 的 view，不過它並不知道該顯示哪個 SwiftUI view，因此待會我們必須從程式設定。

      ![image-20220703172817873](.\099storyboard中嵌入swiftUI.assets\image-20220703172817873.png)

      

   2. 利用 segue 切換頁面

      在 storyboard 裡我們可以使用 segue 切換頁面，直接從 button 連 segue 到 UIHostingController。

      ![image-20220703173052187](.\099storyboard中嵌入swiftUI.assets\image-20220703173052187.png)

   3. ### 利用 **IBSegueAction** 設定 UIHostingController 顯示的 SwiftUI view

      segue 頁面切換時將觸發 IBSegueAction function，此 function 將回傳 segue 終點連接的 controller，在我們的例子裡將是 UIHostingController，因此我們可定義此 function，在產生 UIHostingController 時設定它顯示的 SwiftUI view。

      ```swift
      
      import SwiftUI
      
      @IBSegueAction func showLyrics(_ coder: NSCoder) -> UIViewController? {
         return UIHostingController(coder: coder, rootView: LyricsView())
      }
      ```

      # 利用 Container View 容納 SwiftUI view

      SwiftUI view 不一定要霸道地佔滿整個螢幕，我們也可以利用 Container View 將 SwiftUI view 變成 view controller 裡某個區塊的畫面。

      ![image-20220703173641709](.\099storyboard中嵌入swiftUI.assets\image-20220703173641709.png)

程式和剛剛一樣，只是現在變成從 Container View 的 segue 拉線定義 IBSegueAction function。

```swift
@IBSegueAction func showLyrics(_ coder: NSCoder) -> UIViewController? {   UIHostingController(coder: coder, rootView: LyricsView())}
```



# 從 view controller 的程式切換到 SwiftUI view

學會用 UIHostingController 容納 SwiftUI view 後，我們也可以不依靠 segue，直接從程式生成 UIHostingController，然後呼叫 present(_:animated:completion:)，pushViewController(_:animated:)，show(_:sender:) & addChild(_:) 顯示。

- 例子 1: 呼叫 present(_:animated:completion:)

```swift
@IBAction func showLyrics(_ sender: Any) {   let controller = UIHostingController(rootView: LyricsView(name: "可有可無"))   present(controller, animated: true, completion: nil)}
```

- 例子 2: 呼叫 pushViewController(_:animated:)

```swift
@IBAction func showLyrics(_ sender: Any) {
   let controller = UIHostingController(rootView: LyricsView(name: "可有可無"))
      
   self.navigationController?.pushViewController(controller, animated: true)
}
```