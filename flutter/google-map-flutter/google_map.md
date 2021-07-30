### 1. Get an API key 
Get an API key at https://cloud.google.com/maps-platform/.
https://console.cloud.google.com/

![](img/2020-10-08-19-50-25.png)

すべての Cloud Platform プロダクトへのアクセス
アプリや、ウェブサイト、サービスの構築と実行に必要な Firebase や Google Maps API などがすべて使用できます。

分别设置map api key 对于android和ios有效

![](img/2020-10-26-16-12-32.png)



### 2. 将 API key 加入项目中
将下记加入到android/app/src/main/AndroidManifest.xml文件的manifest --> application 节点下
```
<meta-data
            android:name="com.google.android.geo.API_KEY"
            android:value="AIzaSyBmCIH5XTgUL8o7RqNIVdol10-bTztwbi8"/>
```
![](img/2020-10-08-21-57-10.png)

**IOS設定**

ios/Runner/AppDelegate.swiftに以下を追記します
```dart
import UIKit
import Flutter

@UIApplicationMain
@objc class AppDelegate: FlutterAppDelegate {
  override func application(
    _ application: UIApplication,
    didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
  ) -> Bool {
    /* Google地図apiアクセスkey設定 */
    GMSServices.provideAPIKey("AIzaSyBmCIH5XTgUL8o7RqNIVdol10-bTztwbi8")
    GeneratedPluginRegistrant.register(with: self)
    return super.application(application, didFinishLaunchingWithOptions: launchOptions)
  }
}
```
![](img/2020-10-23-18-52-05.png)

![](img/2020-10-23-18-56-36.png)


ios/Runner/Info.plistを開いて以下を追記します
![](img/2020-10-23-18-54-45.png)


### 3. 显示地图 

1. flutter 项目中引入引入google_maps_flutter plugin
`google_maps_flutter_platform_interface: ^1.0.4`

![](img/2020-10-08-20-35-55.png)

2. 显示地图
只要设置一个初始经纬度，然后创建一个GoogleMap widget实例，设置到画面上就可以了。
* 初始经纬度
    ```dart
    // 浅草桥经纬度
    _initPosition = CameraPosition(
          target: LatLng(35.7007442, // 纬度
            139.7850068), // 经度
          zoom: 14.4746,
        );

    ```
* 创建一个GoogleMap
  ```dart
    @override
  Widget build(BuildContext context) {

    // 异步获取当前位置
    _updateMap();

    // 如果谷歌地图没有创建，显示一个空容器
    return new Center(
      cild: GoogleMap(
      // mapType: MapType.normal,
        myLocationEnabled: true,
        // 是否显示当前位置button
        myLocationButtonEnabled: false,
        // 初始显示位置不能为空
        initialCameraPosition: _initPosition,
    );

  }
  ```

###　3．flutter获取地理位置使用Geolocator插件获取用户当前位置
1. 引入插件 Geolocator
    ` geolocator: ^6.0.0`

2. 添加 Android权限
将下记加入到android/app/src/main/AndroidManifest.xml文件的manifest节点下
```
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
```

![](img/2020-10-09-11-00-43.png)

IOS
info.plistの追記部分を抜粋
```
<key>NSLocationAlwaysUsageDescription</key>
    <string>Your location is required for this app</string>
    <key>NSLocationWhenInUseUsageDescription</key>
    <string>Your location is required for this app</string>
    <key>NSLocationAlwaysAndWhenInUseUsageDescription</key>
    <string>Your location is required for this app</string>
```

3. 获取现在位置调用geolocator.dart文件的getCurrentPosition函数

获取现在位置只能用异步的方式，而且这个函数会不停的调用回调函数刷新当前位置。
```
 getCurrentPosition(desiredAccuracy: LocationAccuracy.best)
        .then((Position position) {
        print(position.latitude);
        print(position.longitude);
        _currentPosition = CameraPosition(
          target: LatLng(position.latitude, position.longitude),
          zoom: 14.4746,
        );
        // 下面更新地图组件重绘画面
      }
    ).catchError((e) {
      print(e);
    });
```

### 4. 迁移到指定位置
通过谷歌地图控制器可以对地图进行操作
1. 声明谷歌地图控制器
```
// 谷歌地图控制器
  Completer<GoogleMapController> _controller = Completer();
```
2. 迁移到指定位置
```
  Future<void> _goToPosion(CameraPosition position) async {
    final GoogleMapController controller = await _controller.future;
    controller.animateCamera(CameraUpdate.newCameraPosition(position));
  }
```

### 5. 设定mark



### 5. 完整代码

```dart
import 'package:flutter/material.dart';
import 'package:google_maps_flutter/google_maps_flutter.dart';
import 'dart:async';
import 'package:geolocator/geolocator.dart';

class SHopMapPage extends StatelessWidget  {
  const SHopMapPage({Key key}) : super(key: key);

  static const TextStyle optionStyle =
  TextStyle(fontSize: 30, fontWeight: FontWeight.bold);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('shop Map page'),
      ),
      body: SHopMap(),
   );
  }
}

class SHopMap extends StatefulWidget {
  @override
  State<SHopMap> createState() => SHopMapState();
}

class SHopMapState extends State<SHopMap> {

  // 谷歌地图控制器
  Completer<GoogleMapController> _controller = Completer();
  // 谷歌地图widget
  GoogleMap _map ;
  // 谷歌地图地图显示类型
  MapType _mapType = MapType.normal;
  // 当前位置
  CameraPosition _currentPosition;

  /*
  * 根据当前位置，创建谷歌地图widget
  * */
  void _createMap() {
    // 创建谷歌地图widget
    _map = GoogleMap(
      // mapType: MapType.normal,
        myLocationEnabled: true,
        // 是否显示当前位置button
        myLocationButtonEnabled: false,
        // 初始显示位置不能为空
        initialCameraPosition: _currentPosition,
        // map创建成功后的回调函数，用来得到GoogleMapController
        onMapCreated: (GoogleMapController controller) {
          _controller.complete(controller);
        });
  }

  /*
  * 异步获取当前位置更新地图到当前位置
  * */
  _updateMap() {
    getCurrentPosition(desiredAccuracy: LocationAccuracy.best)
        .then((Position position) {
      if(_currentPosition == null) {
        _currentPosition = CameraPosition(
          target: LatLng(position.latitude, position.longitude),
          zoom: 14.4746,
        );
        // 创建地图组件，调用setState,更新重绘画面
        setState(() {
          if(_map == null) {
            _createMap();
          } else {
            _goToPosion(_currentPosition);
          }
        });
      }
    }).catchError((e) {
      print(e);
    });
  }


  // 当悬浮button按下，迁移到当前位置
  _move2CurrentLocation() {
    _currentPosition = null;
    _updateMap();
  }

  @override
  Widget build(BuildContext context) {

    // 异步获取当前位置
    _updateMap();

    // 如果谷歌地图没有创建，显示一个空容器
    return new Scaffold(
      body: _map==null ? Container() : _map,
      // 悬浮button
      floatingActionButton: FloatingActionButton.extended(
        onPressed: _move2CurrentLocation,
        label: Text(''),
        icon: Icon(Icons.location_searching_rounded),
      ),
    );

  }
  
  /*
  * 根据参数更改地图显示位置
  * */
  Future<void> _goToPosion(CameraPosition position) async {
    final GoogleMapController controller = await _controller.future;
    controller.animateCamera(CameraUpdate.newCameraPosition(position));
  }
}
```

### 经度,纬度
lat:是Latitude的简写,表示纬度
lng:是Longtitude的简写,表示经度

###　通过检索获得经纬度
google.maps.Geocoder()的.geocode({ 'address': address},geoFunc);

### 参考doc
https://github.com/flutter/plugins/tree/master/packages/google_maps_flutter/google_maps_flutter


### 可能遇到的问题

![](img/2020-10-26-16-10-15.png)

**あなたのAPIコンソールで "Maps SDK for iOS"を有効にしていること**を確認してください " https://console.cloud.google.com/google/maps-apis/ ......"

![](img/2020-10-26-16-12-32.png)

