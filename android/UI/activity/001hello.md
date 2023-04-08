# hello app

## project 结构

默认视图为android，为了更清楚的看到目录结构切换到project视图

![image-20220627131201902](.\001hello.assets\image-20220627131201902.png)

- .grade .idea

  自动生成的文件，不要手动编辑修改

- `app`

  应用程序的代码，资源，主要工作的目录

- build

  编译时自动生成的文件

- gradle

  gradle wrapper的配置文件

- .gitignore

  git忽略文件

- `build.gradle`

  全局的gradle构建脚本，通常不需要修改。

  指定了所用仓库

  指定了android gradle插件（gradle只是一种构建脚本并不是android专用还可用于java，c++编译build）

  ```groovy
  // Top-level build file where you can add configuration options common to all sub-projects/modules.
  buildscript {
      ext.kotlin_version = "1.3.72"
      // 指定仓库
      repositories {
          google()
          jcenter()
      }
      dependencies {
          classpath "com.android.tools.build:gradle:4.1.1" // android gradle插件
          classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"
  
          // NOTE: Do not place your application dependencies here; they belong
          // in the individual module build.gradle files
      }
  }
  
  allprojects {
      repositories {
          google()
          jcenter()
      }
  }
  
  task clean(type: Delete) {
      delete rootProject.buildDir
  }
  ```

  

- gradle.properties

  全局的gradle配置文件，这里的配置属性会影响到项目中所有的gradle构建脚本

- gradlew gradlew.bat

  执行gradle命令的脚本文件，gradlew.bat为windows脚本

- local.properties

  指定android sdk路径，通常会自动设置

- settings.gradle

  指定项目中所有引入的模块，通常会自动引入。

- External Libraries

  android引用的lib文件

### app 结构

app的目录结构

![image-20220627134203855](.\001hello.assets\image-20220627134203855.png)

![image-20220627134822838](.\001hello.assets\image-20220627134822838.png)

- AndroidManifest.xml

  整个android项目的配置文件，四大组件的使用都要在这个文件注册，还有app需要使用的权限

- java

  代码目录，

- res

  画面的layout，使用的图片，icon，定义的颜色，字符串常量

- build.gradle

  ```groovy
  plugins {
      id 'com.android.application'  // 还可以为com.android.library 区别是application可以直接运行，library必须依附于application运行
      id 'kotlin-android'
  }
  
  android {
      compileSdkVersion 30
      buildToolsVersion "32.0.0"
  
      defaultConfig {
          applicationId "com.example.helloworldapplication"
          minSdkVersion 16
          targetSdkVersion 30
          versionCode 1
          versionName "1.0"
  
          testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
      }
  
      buildTypes {
          release {
              minifyEnabled false // true 为使用混淆
              proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
          }
      }
      compileOptions {
          sourceCompatibility JavaVersion.VERSION_1_8
          targetCompatibility JavaVersion.VERSION_1_8
      }
      kotlinOptions {
          jvmTarget = '1.8'
      }
  }
  
  dependencies {
      // 本地的jar文件，库文件（com.android.library），远程依赖
      implementation "org.jetbrains.kotlin:kotlin-stdlib:$kotlin_version"
      implementation 'androidx.core:core-ktx:1.2.0'
      implementation 'androidx.appcompat:appcompat:1.1.0'
      implementation 'com.google.android.material:material:1.1.0'
      implementation 'androidx.constraintlayout:constraintlayout:1.1.3'
      testImplementation 'junit:junit:4.+'
      androidTestImplementation 'androidx.test.ext:junit:1.1.1'
      androidTestImplementation 'androidx.test.espresso:espresso-core:3.2.0'
  }
  ```

  

## AndroidManifest.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.helloworldapplication">

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.HelloWorldApplication">
        <activity android:name=".MainActivity">
            <intent-filter>
            <!-- 指定launch activity为MainActivity -->
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

</manifest>
```

## 启动流程

1. 首先在AndroidManifest.xml文件中定义了启动Activity

2. 在MainActivity引入画面定义layout文件

   ```kotlin
   class MainActivity : AppCompatActivity() {
       private val TAG = "MainActivity"
       
       override fun onCreate(savedInstanceState: Bundle?) {
           super.onCreate(savedInstanceState)
           // 画面定义ayout文件
           setContentView(R.layout.activity_main)
           Log.d(TAG, "onCreate: ")
       }
   }
   ```

   

3. 在layout文件中，加入UI组件TextView

   ![image-20220627141842590](D:\github\knowhow\android_studio\android_native\001hello.assets\image-20220627141842590.png)

   ```xml
   <?xml version="1.0" encoding="utf-8"?>
   <androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
       xmlns:app="http://schemas.android.com/apk/res-auto"
       xmlns:tools="http://schemas.android.com/tools"
       android:layout_width="match_parent"
       android:layout_height="match_parent"
       tools:context=".MainActivity">
   
       <TextView
           android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:text="@string/hello_str"
           app:layout_constraintBottom_toBottomOf="parent"
           app:layout_constraintLeft_toLeftOf="parent"
           app:layout_constraintRight_toRightOf="parent"
           app:layout_constraintTop_toTopOf="parent" />
   
   </androidx.constraintlayout.widget.ConstraintLayout>
   ```

4. 在资源res/values/strings文件中定义hello_str

   ```xml
   <resources>
       <string name="app_name">HelloWorldApplication</string>
       <string name="hello_str">Hello World!...</string>
   </resources>
   ```

   



## 引用资源

### 引用string常量

- 在代码中通过

  ```java
  R.string.hello_str
  ```

  

- 在xml文件中

  ```
  @string/hello_str
  ```

## Log logcat

在android中输入logd 按tab键自动补全

```kotlin
Log.d("MainActivity", "onCreate: ") //第一个参数为组件（用于过滤用），第一个参数为message
```

