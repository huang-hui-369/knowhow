# 1. install flutter 

## 1.1 Get the Flutter SDK

1. Download the following installation bundle to get the latest stable release of the Flutter SDK
2. Extract the zip file and place the contained `flutter` in the desired installation location for the Flutter SDK (for example, `C:\Users\<your-user-name>\Documents`).

or  get source

```shell
C:\src>git clone https://github.com/flutter/flutter.git -b stable
```



## 1.2 add flutter bin path to path env  variables 

## 1.3 run flutter doctor

```shell
d:\>flutter doctor
Doctor summary (to see all details, run flutter doctor -v):
[√] Flutter (Channel stable, 1.20.4, on Microsoft Windows [Version 10.0.19042.1466], locale ja-JP)
[X] Android toolchain - develop for Android devices
    X Unable to locate Android SDK.
      Install Android Studio from: https://developer.android.com/studio/index.html
      On first launch it will assist you in installing the Android SDK components.
      (or visit https://flutter.dev/docs/get-started/install/windows#android-setup for detailed instructions).
      If the Android SDK has been installed to a custom location, set ANDROID_SDK_ROOT to that location.
      You may also want to add it to your PATH environment variable.

[!] Android Studio (not installed)
[!] Connected device
    ! No devices available

! Doctor found issues in 3 categories.
```



# 2. Install Android Studio

## 2.1 Install Android Studio

## 2.2 Start Android Studio

## 2.3 flutter config --android-studio-dir <directory>

```shell
D:\>flutter config --android-studio-dir D:\program-files\android\Android Studio
Setting "android-studio-dir" value to "D:\program-files\android\Android".

You may need to restart any open editors for them to read new settings.

D:\>flutter config --android-sdk D:\program-files\android\sdk
Setting "android-sdk" value to "D:\program-files\android\sdk".

You may need to restart any open editors for them to read new settings.
```



## 2.4 Set up the Android emulator

1. Launch Android Studio , click the AVD Manager icon, and select Create Virtual Device…
   - In older versions of Android Studio, you should instead launch **Android Studio > Tools > Android > AVD Manager** and select **Create Virtual Device…**. (The **Android** submenu is only present when inside an Android project.)
   - If you do not have a project open, you can choose **Configure > AVD Manager** and select **Create Virtual Device…**
2. Choose a device definition and select **Next**.
3. Select one or more system images for the Android versions you want to emulate, and select **Next**. An *x86* or *x86_64* image is recommended.
4. Under Emulated Performance, select **Hardware - GLES 2.0** to enable [hardware acceleration](https://developer.android.com/studio/run/emulator-acceleration).
5. Verify the AVD configuration is correct, and select **Finish**.

## 2.5 Enable [VM acceleration](https://developer.android.com/studio/run/emulator-acceleration) on your machine

1. Windows Hypervisor Platform を使用する VM アクセラレーションを設定する

   WHPX を有効にするには、コンピュータが次の要件を満たしている必要があります。

   - Intel プロセッサ: Virtualization Technology（VT-x）、Extended Page Tables（EPT）、Unrestricted Guest（UG）機能のサポート。コンピュータ BIOS 設定で VT-x を有効にする必要があります

   - Android Studio 3.2 Beta 1 以降（[developer.android.com](https://developer.android.com/studio/preview) からダウンロード）

   - Android Emulator バージョン 27.3.8 以降（[SDK Manager](https://developer.android.com/studio/intro/update#sdk-manager) を使用してダウンロード）

     

2. Windows に WHPX をインストールするには

   1. Windows デスクトップで、Windows アイコンを右クリックし、[**アプリと機能**] を選択します。
   2. [**関連設定**] で、[**プログラムと機能**] をクリックします。
   3. [**Windows の機能の有効化または無効化**] をクリックします。
   4. [Hyper-v] --> [Windows Hypervisor Platform] を選択します。



https://developer.android.com/studio/run/emulator-acceleration#vm-windows

## 2.6 Agree to Android Licenses

```shell
D:\>flutter doctor --android-licenses
Warning: File C:\Users\huanghui\.android\repositories.cfg could not be loaded.
All SDK package licenses accepted.======] 100% Computing updates...
```



## 2.7  Install the Flutter and Dart plugins 

1. Open plugin preferences (**File > Settings > Plugins**).
2. Select **Marketplace**, select the Flutter plugin and click **Install**.

# 3. Run the app





#   參照文檔

https://docs.flutter.dev/get-started/install/windows