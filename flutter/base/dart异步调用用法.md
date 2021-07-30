# initState中调用多个异步方法

像如下这样
_getCurrentPosition中取得当前位置，然后调用setState更新画面。
_buildMarkers中取得当前位置附件的店铺Marker，然后调用setState更新画面。

### 错误方法
像如下这样，只会取得当前位置，_buildMarkers中调用的异步函数shopBl.getShopMarker根本不会执行，也不会返回结果，。

```dart
 @override
  void initState() {
    /// 调用异步方法1  
    _getCurrentPosition();
    /// 调用异步方法2  
    _buildMarkers()
    super.initState();
  }

  void _buildMarkers() {
      /// 调用异步方法  
      markerList = shopBl.getShopMarker(中心点纬度，中心点经度，距离);
  }

  ```

### 正确方法1
要同步的调用_getCurrentPosition和_buildMarkers
```dart
 @override
  void initState() {
    /// 调用异步方法1  
    await _getCurrentPosition();
    /// 调用异步方法2  
    await _buildMarkers()

    super.initState();
  }

  void _buildMarkers() {
      /// 调用异步方法  
      markerList = shopBl.getShopMarker(中心点纬度，中心点经度，距离);
      setState();
  }
  ```

### 正确方法2
```dart
 @override
  void initState() {
    /// 调用异步方法1  
    _getCurrentPosition().then((value) {
        /// 设置取得数据, 更新画面

    }
       

    ).catchError((){
        /// error处理
    }). whenComplete(() {
        /// finally处理
         /// 调用异步方法2  
        _buildMarkers()
    }

    );
   
    super.initState();
  }

  ```