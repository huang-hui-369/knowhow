# 大体流程
1. 在apple developer网站注册一个apple developer id
2. 为app创建a unique Bundle ID（每一个app有一个唯一标识）
3. 为app创建an application record
4. 
5. 在xcode中设置apple developer id和Bundle ID
6. build package ipa
7. 在xcode的menu中选择product-->atchive 生成ipa上传到app store
8. 在apple store connect中追加测试手机的UDID
9. 追加测试人员 apple id
10. 在apple store connect中选择上传的app的版本到test flight中
11. 在手机中通过Apple store安装test flight
12. 通过test flight就可以安装app
13. 测试通过后可以选择版本，直接提交审查

## 1. 注册一个apple developer id
1. 可以在 [apple developer网站](https://developer.apple.com/jp/programs/enroll/)上注册一个，需要每年99美元

![](img\2021-03-01-12-43-20.png)

2. 填写membership信息 ？
   是在这里填写还是需要登录以后填写
   ![](img\2021-03-01-13-30-49.png)

3. 点击 [登录开始]，输入信息，按照提示完成(https://developer.apple.com/enroll/identity/edit)

## 2. 创建a unique Bundle ID
每一个ios app 都关联着唯一的一个Bundle ID
1. 打开你的developer account的 [App IDs page](https://developer.apple.com/account/ios/identifier/bundle)
2. 点击左侧菜单 Certificates, Identifiers & Profiles
![](img\2021-03-01-12-37-43.png)
3. 点击左侧的Identifiers link
   ![](img\2021-03-01-13-15-09.png)
4. 点击右侧的➕号，创建一个新的Bundle ID
   ![](img\2021-03-01-13-18-19.png)
5. 选择Bundle ID的类型，一般选择 app id，然后点击 continue link
   ![](img\2021-03-01-13-17-53.png)
6. 选择 app 类型 然后点击 continue link
   ![](img\2021-03-01-13-20-25.png)
7. 输入an app name, 输入 Explicit App ID, Select the services your app uses
   ![](img\2021-03-01-13-39-57.png)
8.  On the next page, confirm the details and click Register to register your Bundle ID.
   ![](img\2021-03-01-13-40-57.png)

## 3. 为app创建an application record
Register your app on App Store Connect:
1. Open [App Store Connect](https://appstoreconnect.apple.com/) in your browser.
   ![](img\2021-03-01-13-47-10.png)
2. On the App Store Connect landing page, click My Apps.
   ![](img\2021-03-01-13-47-55.png)
3. Click + in the top-left corner of the My Apps page, then select New App.
   ![](img\2021-03-01-13-48-20.png)
4. Fill in your app details in the form that appears. In the Platforms section, ensure that iOS is checked. Since Flutter does not currently support tvOS, leave that checkbox unchecked. Click Create.
   ![](img\2021-03-01-13-49-52.png)
5. Navigate to the application details for your app and select App Information from the sidebar.
6. In the General Information section, select the Bundle ID you registered in the preceding step.

## 4. 设置Xcode project
Xcode project是由flutter自动生成的。
<font color='red'>Xcode版本一定>12</font>

1. 在Xcode打开ios/Runner.xcworkspace
   打开后会出现Runner和Pod两个project

2. 设置schema
   ![](img\2021-03-01-15-35-49.png)


3. 设置bundle id
   点击xcode左上角的show project navigator -->
   选择 Runner project  -->
   选择 target Runner -->
   选择 General tab，输入bundle id
    ![](img\2021-03-01-14-39-10.png)

4. 设置签名证书方法 auto manage signning
   选择 Signing & Capabilities tab
   给Automatically manage signing 打钩
   ![](img\2021-03-01-14-42-57.png)
   设置team
   点击 Add an Account
   ![](img\2021-02-26-19-07-11.png)
   输入 Apple ID 和密码 会自动带出 team id等信息
   ![](img\2021-02-26-19-07-31.png)

## 5. build package ipa 方法一 通过xcode
  先执行flutter build ios（会在pod目录下生成需要的第三方库文件，如果不执行会出错）
  再用xcode 生成 ipa

1. flutter build app
   ```
   cd /Users/huang/work/GUT/msp/coding/msp_app
   flutter build ios --release
   ```
   成功后会生成  /Users/huang/work/GUT/msp/coding/msp_app/build/ios/iphoneos/Runner.app

2. xcode build ipa
   选中需要build的project
   在xcode的菜单栏上选择 product --> archive
   第一次会自动生成 a provisioning profile，证明书，和自动签名

   ![](img\2021-02-26-13-55-34.png)

3. distribute app
   生成以后，可以通过点击 Distribute app button来直接发布app 到 Apple store

   ![](img\2021-03-01-10-13-56.png)

   ![](img\2021-03-01-10-14-31.png)

   ![](img\2021-03-01-10-15-06.png)

   ![](img\2021-03-01-10-15-29.png)

   ![](img\2021-03-01-15-23-49.png)

   ![](img\2021-03-01-15-24-32.png)

   ![](img\2021-03-01-15-27-15.png)

   在 apple store connect中确认发布的app
   
   ![](img\2021-03-01-15-31-43.png)

4. export ipa
   点击 distribute App button，选择 development
   ![](img\2021-03-01-15-37-04.png)

   ![](img\2021-03-01-15-39-06.png)

   ![](img\2021-03-01-15-39-34.png)

   ![](img\2021-02-26-15-01-36.png)

   ![](img\2021-02-26-15-01-52.png)



## 5. build package ipa 方法二 通过flutter 命令
   flutter 1.24以上版本支持直接执行以下命令生成 ipa
   flutter build ipa
   <font color='red'>必须注意的是，在没有provisioning profile的时候
   **会报出以下错误**
   Encountered error while building IPA:
   error: exportArchive: "Runner.app" requires a provisioning profile.
   **解决办法**
   必须先通过xcode build一次后，就会生成所需的provisioning profile
   </font>

1. 更新flutter 版本 需要 1.24以上版本
   ```
   # 确认版本
   flutter --version
   # 确认当前channel
   flutter channel 
   # 切换到 beta channel 
   flutter channel beta
   # 更新flutter版本
   flutter upgrade
   # build ios
   cd /Users/huang/work/GUT/msp/coding/msp_app
   flutter build ios --release
   ```
   ![](img\2021-03-01-14-54-04.png)

   ![](img\2021-03-01-14-54-42.png)


2. 执行命令 flutter build ipa
   ```
   flutter build ipa
   ```
   这样就会生成压缩包Runner.xdarchive
   ![](img\2021-03-01-15-07-30.png)
   
    并没有生成ipa，如果要生成ipa必须提供文件
    ExportOptions.plist
    **这里重要的是teamID**
    ```
    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
    <plist version="1.0">
      <dict>
        <key>compileBitcode</key>
        <true/>
        <key>destination</key>
        <string>export</string>
        <key>method</key>
        <string>development</string>
        <key>signingStyle</key>
        <string>automatic</string>
        <key>stripSwiftSymbols</key>
        <true/>
        <key>teamID</key>
        <string>26WL9HQ9BD</string>
        <key>thinning</key>
        <string>&lt;none&gt;</string>
      </dict>
    </plist>
    ```

  重新生成ipa
  ```
  flutter build ipa --export-options-plist=ExportOptions.plist
  ``` 

#　可能遇到到问题

## The iOS deployment target 'IPHONEOS_DEPLOYMENT_TARGET' is set to 8.0

![](img\2021-02-26-12-25-42.png)

**解决办法**
* Set MinimumOSVersion to 9.0 in ios/Flutter/AppFrameworkInfo.plist
  ![](img\2021-02-26-12-35-52.png)

* Ensure that you uncomment platform :ios, '9.0' in ios/Podfile
* Ensure that ios/Podfile contains the following post install script:
  ```
  post_install do |installer|
      installer.pods_project.targets.each do |target|
        flutter_additional_ios_build_settings(target)
        target.build_configurations.each do |config|
          config.build_settings['IPHONEOS_DEPLOYMENT_TARGET'] = '9.0'
        end
      end
    end
  ```
  ![](img\2021-02-26-12-39-15.png)


## flueer.h头文件找不到

执行 flutter clean

## 头文件找不到
![](img\2021-02-26-12-41-36.png)

**解决办法**
先执行 flutter build ios
会生成引用的第三方库所需的pod文件。


## app store connect 

