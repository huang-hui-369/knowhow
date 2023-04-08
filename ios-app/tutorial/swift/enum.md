### Switch statements

Switch statements can match on many types, including ranges

```swift
let macOSVersion: Int = ...
    switch macOSVersion {
    case 0...8:
        print("A big cat")
    case 9...15:
        print("California locations")
    default:
        print("Greetings, people of the future! What's new in 10.\(macOSVersion)?")
    }
```

## enum

```swift
enum PieType {
    case apple
    case cherry
    case pecan
}

let favoritePie = PieType.apple                              apple
let name: String
switch favoritePie {
case .apple:
    name = "Apple"                                           "Apple"
case .cherry:
    name = "Cherry"
case .pecan:
    name = "Pecan"
}

```

### Enumerations and raw values

```swift
enum PieType: Int {
    case apple = 0
    case cherry
    case pecan
}
let pieRawValue = PieType.pecan.rawValue                2

if let pieType = PieType(rawValue: pieRawValue) {
    // Got a valid 'pieType'!
    print(pieType)                                      "pecan\n"
}
```

The raw value for an enum is often an Int, but it can be any integer or floating-point number type as well as the String and Character types