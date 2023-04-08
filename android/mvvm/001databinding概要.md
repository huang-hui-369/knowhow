# databinding

数据绑定库是一种支持库，借助该库，您可以使用声明式编程（而非程序化地）将布局中的界面组件绑定到应用中的数据源。

## 使用流程

### 1. 配置使用数据绑定库

在app的 `build.gradle` 文件中添加 `dataBinding` 元素

```groovy
android {
        ...
        dataBinding {
            enabled = true
        }
    }
```

 **Preview** 窗格显示数据绑定表达式的默认值（如果提供）

```xml
<TextView android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@{user.firstName, default=my_default}"/>
    
```

### 2. 在布局xml文件中使用绑定表达式

在非绑定布局文件中加入根标记 `layout`，后跟 `data` 元素和非绑定布局文件中的`view` 根元素。

```xml
<?xml version="1.0" encoding="utf-8"?>
    <layout xmlns:android="http://schemas.android.com/apk/res/android">
        <!--新加data元素 -->
       <data>
           <variable name="user" type="com.example.User"/>
       </data>
       <!--非绑定布局文件中的view根元素 -->
       <LinearLayout
           android:orientation="vertical"
           android:layout_width="match_parent"
           android:layout_height="match_parent">
           <TextView android:layout_width="wrap_content"
               android:layout_height="wrap_content"
               android:text="@{user.firstName}"/>
           <TextView android:layout_width="wrap_content"
               android:layout_height="wrap_content"
               android:text="@{user.lastName}"/>
       </LinearLayout>
        <!--非绑定布局文件中的view根元素 -->
    </layout>
```

### 3.创建`data` 元素

 例如创建一个plain-old 对象来描述 `User` 实体

```kotlin
data class User(val firstName: String, val lastName: String)
```



### 4.绑定数据

系统会为每个布局文件生成一个绑定类。默认情况下，类名称基于布局文件的名称，它会转换为 Pascal 大小写形式并在末尾添加 Binding 后缀。以上布局文件名为 `activity_main.xml`，因此生成的对应类为 `ActivityMainBinding`。

**activity中的代码**

```kotlin
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        // 获取自动生成的绑定类
        val binding: ActivityMainBinding = DataBindingUtil.setContentView(
                this, R.layout.activity_main)
		// 设置绑定的数据 
        binding.user = User("Test", "User")
    }

```

如果您要在 `Fragment`、`ListView` 或 `RecyclerView` 适配器中使用数据绑定项，您可能更愿意使用绑定类或 [`DataBindingUtil`](https://developer.android.com/reference/androidx/databinding/DataBindingUtil) 类的 [`inflate()`](https://developer.android.com/reference/androidx/databinding/DataBindingUtil#inflate(android.view.LayoutInflater, int, android.view.ViewGroup, boolean, android.databinding.DataBindingComponent)) 方法，如以下代码示例所示：

```kotlin
    val listItemBinding = DataBindingUtil.inflate(layoutInflater, R.layout.list_item, viewGroup, false)

```





## 表达式语言

表达式语言与托管代码中的表达式非常相似。您可以在表达式语言中使用以下运算符和关键字：

- 算术运算符 `+ - / * %`
- 字符串连接运算符 `+`
- 逻辑运算符 `&& ||`
- 二元运算符 `& | ^`
- 一元运算符 `+ - ! ~`
- 移位运算符 `>> >>> <<`
- 比较运算符 `== > < >= <=`（请注意，`<` 需要转义为 `<`）
- `instanceof`
- 分组运算符 `()`
- 字面量运算符 - 字符、字符串、数字、`null`
- 类型转换
- 方法调用
- 字段访问
- 数组访问 `[]`
- 三元运算符 `?:`

示例：

```xml
android:text="@{String.valueOf(index + 1)}"
    android:visibility="@{age > 13 ? View.GONE : View.VISIBLE}"
    android:transitionName='@{"image_" + id}'
```



### Null 合并运算符

如果左边运算数不是 `null`，则 Null 合并运算符 (`??`) 选择左边运算数，如果左边运算数为 `null`，则选择右边运算数。

```xml
android:text="@{user.displayName ?? user.lastName}"
```

### 避免出现 Null 指针异常

生成的数据绑定代码会自动检查有没有 `null` 值并避免出现 Null 指针异常。例如，在表达式 `@{user.name}` 中，如果 `user` 为 Null，则为 `user.name` 分配默认值 `null`。如果您引用 `user.age`，其中 age 的类型为 `int`，则数据绑定使用默认值 `0`。

### 视图引用

表达式可以通过以下语法按 ID 引用布局中的其他视图（exampleText）：

**注意**：绑定类将 ID 转换为驼峰式大小写。

在以下示例中，`TextView` 视图引用同一布局中的 `EditText` 视图：

```xml
<EditText
        android:id="@+id/example_text"
        android:layout_height="wrap_content"
        android:layout_width="match_parent"/>
    <TextView
        android:id="@+id/example_output"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@{exampleText.text}"/>
```

### 集合

为方便起见，可使用 `[]` 运算符访问常见集合，例如数组、列表、稀疏列表和映射。

```xml
<data>
        <import type="android.util.SparseArray"/>
        <import type="java.util.Map"/>
        <import type="java.util.List"/>
        <variable name="list" type="List&lt;String>"/>
        <variable name="sparse" type="SparseArray&lt;String>"/>
        <variable name="map" type="Map&lt;String, String>"/>
        <variable name="index" type="int"/>
        <variable name="key" type="String"/>
    </data>
    …
    android:text="@{list[index]}"
    …
    android:text="@{sparse[index]}"
    …
    android:text="@{map[key]}"
    
```

**注意**：要使 XML 不含语法错误，您必须转义 `<` 字符。例如：不要写成 `List<String>` 形式，而是必须写成 `List<String>`。

### 字符串字面量

您可以使用单引号括住特性值，这样就可以在表达式中使用双引号，如以下示例所示：

```xml
android:text='@{map["firstName"]}'
```

## 事件处理

你可以在xml布局文件中绑定事件处理

### 方法1 方法引用

例如

1. 设定xml布局文件

   ```xml
   <?xml version="1.0" encoding="utf-8"?>
       <layout xmlns:android="http://schemas.android.com/apk/res/android">
          <data>
              <!-- 定义click处理handler -->
              <variable name="handlers" type="com.example.MyHandlers"/>
              <variable name="user" type="com.example.User"/>
          </data>
          <LinearLayout
              android:orientation="vertical"
              android:layout_width="match_parent"
              android:layout_height="match_parent">
              <TextView android:layout_width="wrap_content"
                  android:layout_height="wrap_content"
                  android:text="@{user.firstName}"
                  android:onClick="@{handlers::onClickFriend}"/> <!-- 绑定click处理 -->  
          </LinearLayout>
       </layout>
   ```

2. 创建click handler

   ```kotlin
   class MyHandlers {
           fun onClickFriend(view: View) { ... }
       }
   ```

### 方法2 监听器绑定

监听器绑定是在事件发生时运行的绑定表达式。它们类似于方法引用，但允许您运行任意数据绑定表达式。此功能适用于 Gradle 2.0 版及更高版本的 Android Gradle 插件。

在方法引用中，方法的参数必须与事件监听器的参数匹配。在监听器绑定中，只有您的返回值必须与监听器的预期返回值相匹配（预期返回值无效除外）。例如，请参考以下具有 `onSaveClick()` 方法的 presenter 类：

1. 创建click处理

   ```kotlin
       class Presenter {
           fun onSaveClick(task: Task){}
       }
       
   ```

   

2. 在xml布局文件中 您可以将click事件绑定到创建的click处理`onSaveClick()` 方法，如下所示：

```xml
<?xml version="1.0" encoding="utf-8"?>
    <layout xmlns:android="http://schemas.android.com/apk/res/android">
        <data>
            <variable name="task" type="com.android.example.Task" />
            <!-- 定义click处理handler -->
            <variable name="presenter" type="com.android.example.Presenter" />
        </data>
        <LinearLayout android:layout_width="match_parent" android:layout_height="match_parent">
            <Button android:layout_width="wrap_content" android:layout_height="wrap_content"
            android:onClick="@{() -> presenter.onSaveClick(task)}" /> <!-- 绑定click处理 -->  
        </LinearLayout>
    </layout>
    
```

在表达式中使用回调时，数据绑定会自动为事件创建并注册必要的监听器。当视图触发事件时，数据绑定会对给定表达式求值。与常规绑定表达式一样，在对这些监听器表达式求值时，仍会获得数据绑定的 Null 值和线程安全。

在上面的示例中，我们尚未定义传递给 `onClick(View)` 的 `view` 参数。监听器绑定提供两个监听器参数选项：您可以忽略方法的所有参数，也可以命名所有参数。如果您想命名参数，则可以在表达式中使用这些参数。例如，上面的表达式可以写成如下形式：

```xml
android:onClick="@{(view) -> presenter.onSaveClick(task)}"
    
```

或者，如果您想在表达式中使用参数，则采用如下形式：

```kotlin
    class Presenter {
        fun onSaveClick(view: View, task: Task){}
    }

    
android:onClick="@{(theView) -> presenter.onSaveClick(theView, task)}"
    
```

您可以在 lambda 表达式中使用多个参数：

```kotlin
    class Presenter {
        fun onCompletedChanged(task: Task, completed: Boolean){}
    }

    
<CheckBox android:layout_width="wrap_content" android:layout_height="wrap_content"
          android:onCheckedChanged="@{(cb, isChecked) -> presenter.completeChanged(task, isChecked)}" />
    
```

如果您监听的事件返回类型不是 `void` 的值，则您的表达式也必须返回相同类型的值。例如，如果要监听长按事件，表达式应返回一个布尔值。

```kotlin
    class Presenter {
        fun onLongClick(view: View, task: Task): Boolean { }
    }

    
android:onLongClick="@{(theView) -> presenter.onLongClick(theView, task)}"
    
```

如果由于 `null` 对象而无法对表达式求值，则数据绑定将返回该类型的默认值。例如，引用类型返回 `null`，`int` 返回 `0`，`boolean` 返回 `false`，等等。

如果您需要将表达式与谓词（例如，三元运算符）结合使用，则可以使用 `void` 作为符号。

```xml
android:onClick="@{(v) -> v.isVisible() ? doSomething() : void}"
    
```

#### 避免使用复杂的监听器

监听器表达式功能非常强大，可以使您的代码非常易于阅读。另一方面，包含复杂表达式的监听器会使您的布局难以阅读和维护。这些表达式应该像将可用数据从界面传递到回调方法一样简单。您应该在从监听器表达式调用的回调方法中实现任何业务逻辑。

## 导入、变量和包含

数据绑定库提供了诸如导入、变量和包含等功能。通过导入功能，您可以轻松地在布局文件中引用类。通过变量功能，您可以描述可在绑定表达式中使用的属性。通过包含功能，您可以在整个应用中重复使用复杂的布局。

### 导入

通过导入功能，您可以轻松地在布局文件中引用类，就像在托管代码中一样。您可以在 `data` 元素使用多个 `import` 元素，也可以不使用。以下代码示例将 `View` 类导入到布局文件中：

```xml
<data>
        <import type="android.view.View"/>
        <import type="com.example.MyStringUtils"/>
    </data>
    
```

导入 `View` 类可让您通过绑定表达式引用该类。以下示例展示了如何引用 `View` 类的 `VISIBLE` 和 `GONE` 常量：

```xml
<TextView
       android:text="@{MyStringUtils.capitalize(user.lastName)}"
       android:layout_width="wrap_content"
       android:layout_height="wrap_content"
       android:visibility="@{user.isAdult ? View.VISIBLE : View.GONE}"/>
    
```

#### 类型别名

当类名有冲突时，其中一个类可使用别名重命名。以下示例将 `com.example.real.estate` 软件包中的 `View` 类重命名为 `Vista`：

```xml
<import type="android.view.View"/>
    <import type="com.example.real.estate.View"
            alias="Vista"/>
    
```

您可以在布局文件中使用 `Vista` 引用 `com.example.real.estate.View`，使用 `View` 引用 `android.view.View`。

### 包含

通过使用应用命名空间和特性中的变量名称，变量可以从包含的布局传递到被包含布局的绑定。以下示例展示了来自 `name.xml` 和 `contact.xml` 布局文件的被包含 `user` 变量：

```xml
<?xml version="1.0" encoding="utf-8"?>
    <layout xmlns:android="http://schemas.android.com/apk/res/android"
            xmlns:bind="http://schemas.android.com/apk/res-auto">
       <data>
           <variable name="user" type="com.example.User"/>
       </data>
       <LinearLayout
           android:orientation="vertical"
           android:layout_width="match_parent"
           android:layout_height="match_parent">
           <include layout="@layout/name"
               bind:user="@{user}"/>
           <include layout="@layout/contact"
               bind:user="@{user}"/>
       </LinearLayout>
    </layout>
    
```

数据绑定不支持 include 作为 merge 元素的直接子元素。例如，以下布局不受支持：

```xml
<?xml version="1.0" encoding="utf-8"?>
    <layout xmlns:android="http://schemas.android.com/apk/res/android"
            xmlns:bind="http://schemas.android.com/apk/res-auto">
       <data>
           <variable name="user" type="com.example.User"/>
       </data>
       <merge><!-- Doesn't work -->
           <include layout="@layout/name"
               bind:user="@{user}"/>
           <include layout="@layout/contact"
               bind:user="@{user}"/>
       </merge>
    </layout>
```

# 参考文档

官方 https://developer.android.com/topic/libraries/data-binding