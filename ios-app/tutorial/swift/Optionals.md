Swift types can be optional, which is indicated by appending `?` to a type name

```swift
var anOptionalFloat: Float?
var anOptionalArrayOfStrings: [String]?
var anOptionalArrayOfOptionalStrings: [String?]?

```

You are going to try out two ways of unwrapping an optional variable

1. To forcibly unwrap an optional, you append a ! to its name

   ```swift
   var reading1: Float?                                         nil
   var reading2: Float?                                         nil
   var reading3: Float?                                         nil
   reading1 = 9.8                                               9.8
   reading2 = 9.2                                               9.2
   reading3 = 9.7                                               9.7
   let avgReading = (reading1! + reading2! + reading3!) / 3
   ```

2. A safer way to unwrap an optional is optional binding. Optional binding works within a conditional `if`-`let` statement

   ```swift
   if let r1 = reading1,
       let r2 = reading2,
       let r3 = reading3 {
           let avgReading = (r1 + r2 + r3) / 3
           print(avgReading)
   } else {
       let errorString = "Instrument reported a reading that was nil."
       print(errorString)
   }
   ```

## 使用有可能为nil的类型，例如字典

```swift
let nameByParkingSpace = [13: "Alice", 27: "Bob"]   
if let space13Assignee = nameByParkingSpace[13] {
    print(space13Assignee)
}   
```