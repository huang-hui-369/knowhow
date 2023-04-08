Swift types can be arranged into three basic groups: **structures**, **classes**, and **enumerations**  All three can have:

- properties – values associated with a type
- initializers – code that initializes an instance of a type
- instance methods – functions specific to a type that can be called on an instance of that type
- class or static methods – functions specific to a type that can be called on the type itself

Swift 的structs和enums,比大多数语言都强大得多。除了支持属性、初始化器和方法外，它们还可以conform to protocols and can be extended

Swift 对典型的“原始”类型（例如数字和布尔值）的实现可能会让您感到惊讶：它们都是结构。事实上，所有这些 Swift 类型都是结构

| Numbers:     | Int, Float, Double                                           |
| ------------ | ------------------------------------------------------------ |
| Boolean:     | Bool                                                         |
| Text:        | String, Character                                            |
| Collections: | Array<Element>, Dictionary<Key:Hashable,Value>, Set<Element:Hashable> |

This means that standard types have properties, initializers, and methods of their own. They can also conform to protocols and be extended.

Finally, a key feature of Swift is **optionals**. An optional allows you to store either a value of a particular type or no value at all.

