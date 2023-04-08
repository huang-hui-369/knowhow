## 什么是Data Binding

Data Binding，即数据绑定，是Android团队实现MVVM架构的一种方法，使得数据（对象）可以直接绑定到布局的xml中，数据的变化直接反映到View上。

数据（对象）与View进行双向绑定，所以开发时只需要关注数据（对象）即可，无需关心View的各种繁杂操作（如setVisibility()，setText()等）

## 配置工程的Gradle

加入完成后，然后点击Sync Now，完成后就可以使用Data Binding强大的功能了。

```groovy
android {
    …
    dataBinding {
        enabled = true
    }
}
```

## 修改activity布局

在布局的最外层加入”<layout>”标签即可

修改前activty_main.xml的布局：

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:gravity="center"
    >

    <TextView
        android:id="@+id/tv_example"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textSize="16sp"
        />

</LinearLayout>
```

修改后activty_main.xml的布局：

这里的<data>标签中的元素放的是页面所需的数据

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <data>

    </data>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical"
        android:gravity="center"
        >

        <TextView
            android:id="@+id/tv_example"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:textSize="16sp" />

    </LinearLayout>
</layout>
```

## 在Activity中进行绑定

上面我们修改了布局后，下面我们就可以在代码中进行数据绑定了。此时工程中会自动生成该布局对应的java绑定文件:ActivityMainBinding。仔细观察就会发现，这个文件名就是将布局的下划线形式转换成java规范的驼峰形式，后面加上Binding。

下面进入MainActivity.java中下面我们进行数据绑定操作。

1. 定义成员变量：

```java
private ActivityMainBinding mBinding;
```

1. 在onCreate()中初始化mBinding

```java
@Override
protected void onCreate(@Nullable Bundle savedInstanceState) {
	super.onCreate(savedInstanceState);
    mBinding = DataBindingUtil.setContentView(this, R.layout.activity_main);
    mBinding.tvExample.setText("Binding Text");
}
```

此时就可以从mBinding中获取到布局中的所有View了，比如上面的mBinding.tvExample。

## 在<data>标签中定义界面使用的变量

在这个标签中，我们通常用来做下面的事情：

- 定义所绑定的数据的名称（变量名）及对应类型
- 引入页面所需的类

示例如下：

```xml
<data>
    <import type="android.view.View" />
    <import type="android.text.TextUtils" />
    
    <variable name="visible" type="boolean"/>
    <variable name="title" type="String"/>
    <variable name="user" type="cn.examplecode.androiddatabinding.User"/>
</data>
```

其中”<import/>”标签表示引入一个类，比如上例中引入了View类和一个工具类TextUtils，当然也可以引入你自己的类，比如常量类或者工具类。

下面”<variable/>”标签定义了本页面所需要的各种数据名称或类型，其类型可以是java中的基础类型，或者自定义的类。

## 在布局中使用这些数据

数据设置完毕以后就可以在页面中使用这些数据了，使用起来也非常方便，比起在java代码中操作，可以省去不少代码。

```xml
<TextView
    android:id="@+id/tv_title"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="@{title}"
    android:visibility="@{visible ? View.VISIBLE : View.GONE}"
    />
    
<TextView
    android:id="@+id/tv_username"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="@{user.name}"
    />
```



## 在activity中通过Binding对象设置数据

上面定义了页面中所需要的数据后，下面就需要通过获取到的Binding对象设置这些数据：

```
mBinding.setVisible(true);
mBinding.setTitle("用户信息");
User user = new User();
user.setName("Steve Jobs");
mBinding.setUser(user);
```

这里的setXXX()方法也是IDE自动根据<data>标签中的定义自动生成的。



## 根据View获取Binding

经过前面的学习，我们可以很方便地从数据绑定对象(Binding) 中获取到其绑定的View，但是我们会碰到一些应用场景，比如我们在操作View的时候，需要取得其绑定的Binding对象，以获取到当前View中的子View的引用，从而避免类似findViewById()这样的操作，并且也可以获取到当前View所绑定的数据。

那么该如何从View获取当前绑定的Binding对象呢？

其实很简单，DataBindingUtil就提供了这样的方法，举例如下：

```java
UserBinding userBinding = DataBindingUtil.getBinding(userView);
```

## 在Fragment中的使用

1. 修改布局

   Fragment中使用Data Binding同样需要修改布局，修改方式也跟Activity一样，在最外层加上<layout>标签：

   ```xml
   <?xml version="1.0" encoding="utf-8"?>
   <layout xmlns:android="http://schemas.android.com/apk/res/android"
       xmlns:app="http://schemas.android.com/apk/res-auto">
   
       <data>
   
       </data>
       
       <页面布局.../>
       
   </layout>
   ```

   

2. 在Fragment中进行绑定

   与[在Activity中绑定](https://www.examplecode.cn/2018/07/24/android-databinding-03-activity/)中创建绑定的方式有些不同，但是目的都是获得绑定对象的引用。

   比如我们Fragment的布局文件为：frag_main.xml，具体的方式如下:

   1. 定义成员变量

   ```
   private FragMainBinding mBinding;
   ```

   1. 在onCreateView()中初始化mBinding，并返回View

   ```java
   @Nullable
   @Override
   public View onCreateView(@NonNull LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
       mBinding = FragMainBinding.inflate(inflater);
       mBinding.tvExample.setText("Binding Text");
       return mBinding.getRoot();
   }
   ```

   此时就可以正常操作Binding对象了。

## 使用BindingAdapter

BindingAdapter用来设置布局中View的自定义属性，当使用该属性时，可以自定义其行为。

我们以自定义图片加载的BindingAdapter为例子来说明怎么使用

1. 我们先新建一个ImageBindingAdapter的类， 定义一个imageUrl属性，来下载图片data，显示在ImageView中

   ```java
   public class ImageBindingAdapter {
   
       ("imageUrl")
       public static void bindImageUrl(ImageView view, String imageUrl){
           RequestOptions options =
                   new RequestOptions()
                   .centerCrop()
                   .dontAnimate();
   
           Glide.with(view)
                   .load(imageUrl)
                   .apply(options)
                   .into(view);
       }
   
   }
   ```

   

2. 布局中使用这个属性了

   ```xml
   <ImageView
       android:layout_width="180dp"
       android:layout_height="180dp"
       app:imageUrl="@{user.photo}"
       />
   ```

   

