UUID

https://pub.dev/packages/flutter_udid

Plugin to retrieve a persistent UDID across app reinstalls on iOS and Android.

Getting Started 
import 'package:flutter_udid/flutter_udid.dart';
String udid = await FlutterUdid.udid;

ios 通过keychain获取到的UUID，卸载和在安装不会变。