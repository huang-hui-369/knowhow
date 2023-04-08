```swift
let range = 0..<countingUp.count
for i in range {
    let string = countingUp[i]
    // Use 'string'
}
for string in countingUp {
    // Use 'string'
}
for (i, string) in dicts.enumerated() {
    // (0, "one"), (1, "two")
}

let nameByParkingSpace = [13: "Alice", 27: "Bob"]            [13: "Alice", 27: "Bob"]

for (space, name) in nameByParkingSpace {
    let permit = "Space \(space): \(name)"                   (2 times)
    print(permit)                                            (2 times)
}
```



![image-20221208185001430](D:\github\knowhow\ios-app\tutorial\swift\for-loop.assets\image-20221208185001430.png)