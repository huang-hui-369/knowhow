## 可变变量

关键字是 `var`

```kotlin
var firstName: String
```

## 只读变量

 关键字是`val` 类似于 Java 中的 `final` 关键字,第一次赋值以后，就不可以再赋值了

```kotlin
val firstName: String
```

## 可为null

Kotlin 会明确指定变量能否接受 null 值。其是通过在类型声明后附加"`?`"以进行此项指定。

```kotlin
class User(var firstName: String?, var lastName: String?)
```

## 数据类

`User` 类仅存放数据。对于具有这一角色的类，Kotlin 会提供对应的关键字：`data`。在将此类标记为 `data` 类后，编译器便会自动创建 getter 和 setter。此外，其还会派生 `equals()`、`hashCode()` 和 `toString()` 函数。

```kotlin
data class User(var firstName: String?, var lastName: String?)
```

与 Java 类似，Kotlin 也可拥有一个主构造函数以及一个或多个辅助构造函数。以上示例中的构造函数是 User 类的主构造函数。若您在转换具有多个构造函数的 Java 类，则转换器也会在 Kotlin 中自动创建多个构造函数。构造函数均使用 `constructor` 关键字进行定义。

## 相等性

Kotlin 分为两类相等性：

- 构成相等使用 `==` 运算符，并调用 `equals()` 来确定两个实例是否相等。
- 引用相等使用 === 运算符，以检查两个引用是否指向同一对象。

数据类主构造函数中定义的属性将被用于检查构成相等性。

```kotlin
val user1 = User("Jane", "Doe")
val user2 = User("Jane", "Doe")
val structurallyEqual = user1 == user2 // true
val referentiallyEqual = user1 === user2 // false
```

## 默认参数

在 Kotlin 中，我们可以对函数调用中的参数赋予默认值。当省略参数时，系统便会使用此默认值。在 Kotlin 中，构造函数也属于函数的一种，因此我们可以使用默认参数来将 `lastName` 的默认值指定为 `null`。为此，我们直接将 `null` 赋予 `lastName`。

```kotlin
data class User(var firstName: String?, var lastName: String? = null)

// usage
val jane = User ("Jane") // same as User("Jane", null)
val joe = User ("John", "Doe")
```

## 具名参数

假如 `firstName` 将 `null` 用作其默认值，而 `lastName` 并不如此。在此情况下，由于默认参数将居于未设默认值的参数之前，因此您必须使用具名参数来调用此函数，具体如下：

```kotlin
data class User(var firstName: String? = null, var lastName: String?)

// usage
val jane = User (lastName = "Doe") // same as User(null, "Doe")
val john = User ("John", "Doe")
```

# Null safety

kotlin默认变量不能为null，如果要使变量可为null，必须显示声明 

**类型后面加?表示可为null**

- var b: String?  can be set to null 类型后面加?表示可为null

-  b?.length 如果b为null，return null

- b?.length ?: -1 如果b为null，return -1 

- val parent = node.getParent() ?: return null ， 如果node.getParent() 为null，return null

- ## The !! operator

  ```kotlin
  val l = b!!.length // b为null throw  NPE
  ```

  

- ## Safe casts﻿

  ```kotlin
  val aInt: Int? = a as? Int  // a为null 就不执行cast a 到 Int return null
  ```

- ## Collections of a nullable type﻿

  ```
  val nullableList: List<Int?> = listOf(1, 2, null, 4)
  val intList: List<Int> = nullableList.filterNotNull()
  ```

  

例子

```kotlin
var a: String = "abc" // Regular initialization means non-null by default
a = null // compilation error
var b: String? = "abc" // can be set to null
b = null // ok
print(b) // null

val al = a.length	// ok
val bl = b.length	// compilation error Only safe (?.) or non-null asserted (!!.) calls are allowed on a nullable receiver of type String?
val bl2 = b?.length // null

// 类型后面加?表示可为null
var username: String? = null
println(username)	// null
// 不做处理返回 null
val namelen2 = username?.length
println(namelen2)	// null
 // ?: 为null返回-1
val namelen3 = username?.length ?: -1
println(namelen3)	// -1

// Safe casts
val aInt: Int? = username?.length as? Int
println(aInt)

//抛出空指针异常 The !! operator
val namelength = username!!.length // java.lang.NullPointerException
username = "jack"
println(username)  // jack
println(username?.length)	// 4
println(username?.length ?: -1) // 4 
println(username!!.length) // 4



fun foo(node: Node): String? {
    val parent = node.getParent() ?: return null
    val name = node.getName() ?: throw IllegalArgumentException("name expected")
    // ...
}
```



## Elvis operator

```kotlin
val l = b?.length ?: -1
```



## [字符串模板](https://developer.android.com/codelabs/java-to-kotlin?hl=zh-cn&continue=https%3A%2F%2Fdeveloper.android.com%2Fcourses%2Fpathways%2Fkotlin-for-java%3Fhl%3Dzh-cn%23codelab-https%3A%2F%2Fdeveloper.android.com%2Fcodelabs%2Fjava-to-kotlin#6)

```kotlin
// Java
name = user.getFirstName() + " " + user.getLastName();

// Kotlin
name = "${user.firstName} ${user.lastName}"

var a = 1
// 模板中的简单名称：
val s1 = "a is $a" 
a = 2
// 模板中的任意表达式：
val s2 = "${s1.replace("is", "was")}, but now is $a" // a is 1 but now is 2
```

