# 1 创建秘钥生成签名文件

在 Windows 系统上，
进入flutter项目的/andorid/app目录
执行下面的命令：
```
keytool -genkey -v -keystore ./msp.jks -storetype JKS -keyalg RSA -keysize 2048 -validity 10000 -alias key
```
就会在/andorid/app目录下生成秘钥文件 msp.jks
以后会用到
* 秘钥文件 msp.jks
* alias key
* storePassword 生成秘钥文件时输入
* keyPassword 生成秘钥文件时输入

# 2 编辑build.gradle文件的signingConfigs配置签名

通过编辑 /android/app/build.gradle 文件来为我们的 app 配置签名：
如下追加signingConfigs 节点
```
signingConfigs {
        release{
            storeFile file("msp.jks")
            storePassword "grandunit"
            keyAlias "key"
            keyPassword "grandunit"
        }
        debug{
            storeFile file("msp.jks")
            storePassword "grandunit"
            keyAlias "key"
            keyPassword "grandunit"
        }
     }
```


![](img\2021-01-12-15-25-11.png)

# 3 编辑build.gradle文件的buildTypes
```
buildTypes {
        release {
            // TODO: Add your own signing config for the release build.
            // Signing with the debug keys for now, so `flutter run --release` works.
            signingConfig signingConfigs.release
        }
        debug {
            signingConfig signingConfigs.debug
        }
    }
```

# 4 开启混淆

1. 创建 /android/app/proguard-rules.pro 文件，并添加以下规则：

    ```
    #Flutter Wrapper
    -keep class io.flutter.app.** { *; }
    -keep class io.flutter.plugin.**  { *; }
    -keep class io.flutter.util.**  { *; }
    -keep class io.flutter.view.**  { *; }
    -keep class io.flutter.**  { *; }
    -keep class io.flutter.plugins.**  { *; }
    ```

2. 开启混淆/压缩

    当您使用 Android Gradle 插件 3.4.0 或更高版本构建项目时，该插件不再使用 ProGuard 执行编译时代码优化，而是与 R8 编译器协同工作，处理以下编译时任务：代码缩减（即摇树优化）,资源缩减,混淆,优化
   
   打开 /android/app/build.gradle 文件，定位到 buildTypes 块。

   在 release  配置中将 minifyEnabled  和 useProguard  设为 true，再将混淆文件指向上一步创建的文件。

    ```
    android {

    ...

    buildTypes {

        release {

            signingConfig signingConfigs.release

            minifyEnabled true
            useProguard true

            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'

        }
    }
}
    ```


# 4 生成apk
flutter build apk --release

# 5 安装apk
flutter install

# goolge play store 配布

![](img\2021-03-01-17-33-04.png)

## login 
https://developer.android.com

1. google play クリックする
   ![](img\2021-03-01-16-41-02.png)

2. play　consoleを起動をクリックする
   ![](img\2021-03-01-16-42-40.png)

3. デベロッパー アカウントを選択
   ![](img\2021-03-01-16-43-21.png)

4. デベロッパー ページをクリックする
   ![](img\2021-03-01-16-44-02.png)

5. 情報を入力する
   

# 新规release

可以按顺序从 内部Test -->  close test --> open test --> 制品版release。
也可以支接release制品版

## 1. build apk
1. 修改 pubspec.yaml中的 app的version

    ![](img\2021-03-11-12-17-57.png)

2. build apk
   `flutter build apk --release`

   或者 build appbundle
   `flutter build appbundle --release`

## 2. 上传apk到google play store
1. uploade apk
   点击左侧menu　--> 内部テスト　
   新しいリリースをクリックする

   ![](img\2021-03-11-12-34-04.png)

   ![](img\2021-03-11-12-41-03.png)

   ![](img\2021-03-11-12-40-31.png)

2. upload native debug symbols
   build 完后 进入以下目录
   msp_app\build\app\intermediates\flutter\release
    ![](img\2021-04-14-19-19-08.png)

   将arm64-v8a，armeabi-v7a，x86_64直接打包成symbols.zip
   ![](img\2021-04-14-19-08-54.png)

   上传到play store 
   ![](img\2021-04-14-19-10-02.png)

   ![](img\2021-04-14-19-10-27.png)

   ![](img\2021-04-14-19-11-26.png)
    
   ![](img\2021-03-15-22-51-26.png)
    
   点击上传箭头
   ![](img\2021-04-14-19-12-06.png)

3. 最后到release 制品版
   ![](img\2021-04-14-19-17-57.png)



# google play 自動検出されて問題

##　1．クラッシュ
这是因为google的自动检测不支持flutter，可以忽略。
![](img\2021-03-15-22-50-48.png)

![](img\2021-03-15-22-51-09.png)


 