# 事前准备
1. 在home下创建安装目录
   ```shell
   $ cd ~
   $ mkdir develop
   ```

# 安装jdk 1.8
android studio只能运行在jdk 1.8下，版本高了，低了都不行。
否则会出现 
`Exception in thread "main" java.lang.NoClassDefFoundError: javax/xml/bind/annotation/XmlSchema`

1. [下载jdk1.8](https://www.oracle.com/java/technologies/javase/javase-jdk8-downloads.html)
   下载完后时mac的安装文件jdk-8u271-macosx-x64.dmg，双击后就进行安装
   
2. 安装后目录
   在当前用户目录下的/Library
   ~/Library/Java/JavaVirtualMachines/jdk1.8.0_271.jdk/

3. 设置java_home目录
   
   如果系统上安装了多个版本的jdk，需要运行java_home命令来设定用哪一个版本
   可以用java -version 来确定现在java版本
   
   然后执行命令
   `/usr/libexec/java_home -v 1.8.0_271`
   
   最好设置到.bash_profile中
   ```shell
   export JAVA_HOME=`/usr/libexec/java_home -v 1.8` >> ~/.bash_profile
   source ~/.bash_profile
   ```

# install flutter
flutter 只是一个zip文件，下载完后解压，设置path目录就可以运行了。

1. [download flutter.zip](https://storage.googleapis.com/flutter_infra/releases/stable/macos/flutter_macos_1.22.2-stable.zip)
   下载完后flutter_macos_1.22.2-stable.zip

2. 解压到 $home/develop/flutter目录
   ```shell
   cd ~/development
    unzip ~/Downloads/flutter_macos_1.22.2-stable.zip
   ```
3. 设置flutter bin path
   ```shell
   export PATH="${HOME}/flutter/bin:$PATH" >> ~/.bash_profile
   source ~/.bash_profile
   ```
4. Optionally, pre-download development binaries:
 ```shell
    flutter precache
 ```

# Install Xcode
这个安装完成后，iOS simulator也就安装好了。 


 1. 在apple stroe 中安装，需要 apple id

 2. 安装完后,就可以在lanchpad中打开xcode

 3. 设置 xcode的 apple id
    点击 Xcode --> Preference
   ![](img\2020-10-23-16-34-49.png)
    点击 Accounts ，再点击 + 追加 apple id
   ![](img\2020-10-23-16-35-55.png)

    追加认证证书
    ![](img\2020-10-23-16-38-41.png)

 4. 然后既可以运行  Xcode了
   ![](img\2020-10-23-16-39-52.png)

# Install cocoapods
cocoapods是用来管理xcode ios程序的插件用的，如果你没有用到flutter plugin，可以不用安装，要在xcode安装后来安装

1. 确认ruby 是否安装
   ruby -v

2. 查看安装源地址
   gem sources -l（查看当前镜像地址）

3. 安装 cocoapods
   sudo gem install cocoapods
   安装完成后执行 
   pod setup

# install Android Studio
其实如果不在mac上用Android Studio编写flutter 程序的话是可以不用安装的。如果要安装一定要确认jdk 版本

1. [下载Android Studio](https://developer.android.com/studio)

2. 只要双击Android-Studio.dmg就可以。
    安装完后，Android sdk 就在${HOME}/Library/Android/sdk目录下

3. 设置 .bash_profile
     ```shell
   export PATH="$PATH:/Users/huang/develop/flutter/bin"
   export JAVA_HOME=`/usr/libexec/java_home -v 1.8`
   export ANDROID_SDK_ROOT="/Users/huang/Library/Android/sdk"
   export PATH="$ANDROID_SDK_ROOT/tools/bin"
   export PATH="$PATH:$ANDROID_SDK_ROOT/platform-tools"
   export PATH="$PATH:$ANDROID_SDK_ROOT/tools"
   ```  

4. install flutter dart plugin



# 检证
1. 运行flutter docotr
    
    `flutter docotr`
    
第一次运行，根据提示执行命令，同意android sdk license

![](img\2020-10-23-17-23-48.png)

2. 创建一个demo程序运行在 ios simulator上
     ```shell
     cd ~
     mkdir projects/flutter
     flutter create my_app
     ```
    打开ios 模拟器
    `open -a Simulator`
    运行 myapp
    ```shell
    cd ~/projects/flutter/my_app
    flutter run
    ```
    ![](img\2020-10-23-17-30-01.png)

# xcode 真机调试
1. iphone 通过数据线连接电脑，
2. 打开 xcode，打开新的项目文件，选择 flutter 项目中的ios/Runner.xcworkspace 文件
3. 在 xcode 的 runner 中能够直接选择真机然后运行
直接运行会 build failed，这主要有两个问题
3.1 添加开发者账号
xcode 真机调试是需要 apple 开发者账号，这个和 ios 开发没啥区别
个人申请一个 apple id 作为 personal team ，然后添加 account 即可
3.2 修改 bundle identifier
默认的 bundle identifier 是 com.example.demo，需要改成自己开发者账号下的唯一名字。
之后在 xcode 中运行即可

# 遇到的问题

## 1 Build error - targeted OS version does not support use of thread local variables

打开xcode，双击runner
![](img\2021-01-12-21-16-08.png)

选择general，选择合适的 deployment target 版本 10.3
![](img\2021-01-12-21-16-54.png)

## 2 Xcode keeps showing:-1: SWIFT_VERSION '5.0' is unsupported, supported versions are: 3.0, 4.0, 4.2.Any solution?

打开xcode，双击runner

选择build settings，将 swift 版本 由 5.0 --> 4.2
![](img\2021-01-12-21-20-03.png)

## 3 Automatically assigning platform `iOS` with version `8.0` on target `Runner` because no platform was specified. Please specify a platform for this target in your Podfile.

表面上说提示的是说Podfile没有指定 target 版本

![](img\2021-01-12-21-26-53.png)

实际上是因为runner xcode project的deployment target 版本没有选择对，参见 问题1

## 4 大多数的问题都是因为版本的问题，使用插件的版本，target os版本，swift 语言版本。。。
