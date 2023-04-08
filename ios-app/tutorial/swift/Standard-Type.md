In Xcode, select File → New → Playground

![image-20221208182637000](D:\github\knowhow\ios-app\tutorial\swift\Untitled.assets\image-20221208182637000.png)



```swift
var nextYear: Int = 0                                   // 0
var bodyTemp: Float = 0                                 // 0
var hasPet: Bool = true                                 // true
var arrayOfInts: Array<Int> = []                        // []
var arrayOfStrings: [String] = []                       // []
var dicts: Dictionary<String,String> = [:]               // [:]
var winningLotteryNumbers: Set<Int> = []                // Set([])
let countingUp = ["one", "two"]                         // ["one", "two"]
let nameByParkingSpace = [13: "Alice", 27: "Bob"]       // [13: "Alice", 27: "Bob"]
let availableRooms = Set([205, 411, 412])               // {412, 205, 411}
```

### Properties

```swift
let countingUp = ["one", "two"]                         ["one", "two"]
let secondElement = countingUp[1]                       "two"
countingUp.count                                        2
...
let emptyString = String()                              ""
emptyString.isEmpty                                     true
...
```

### Instance methods

```swift
var countingUp = ["one", "two"]                         ["one", "two"]
let secondElement = countingUp[1]                       "two"
countingUp.count                                        2
countingUp.append("three")                              ["one", "two", "three"]
```

