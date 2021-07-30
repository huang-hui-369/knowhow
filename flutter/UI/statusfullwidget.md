
# statusfull的widget的生命周期

![](img\2020-11-19-15-24-35.png)

即使每次都重新实例一次，会每回调用build，但不会每次都调用 initState()；
是根据key来决定是否每次都实例化。

只要在父widget中调用setState，子widget的didUpdateWidget就一定会被调用，不管父widget传递给子widget构造方法的参数有没有改变。

只要didUpdateWidget被调用，接来下build方法就一定会被调用。

子widget首次被加载时的生命周期：

initState -> build

子widget首次被加载后，如果在父widget中调用setState，子widget的生命周期：

didUpdateWidget -> build


# 问题

## 1. 获取GlobalKey的currentState为null

### 现象 
自己做的组件，设置了GlobalKey _gkey，但是调用_gkey.currentState时，currentState为null。

### 原因 
 当这个组件不在界面显示时，将获取不到currentState， The current state is null if (1) there is no widget in the tree。

**界面1**
![](img\2020-11-19-15-59-14.png)
widget tree1
![](img\2020-11-19-15-57-47.png)

**界面2**
![](img\2020-11-19-16-00-59.png)
![](img\2020-11-19-16-02-10.png)

在界面1中因为 イベント実施有無是出于显示中，可以获取到currentState

在界面2中因为 イベント実施有無是出于显示外，所以获取到currentState为null。

### 解决办法
1. 简单粗暴方法
   在StateFullWidget中保存state，然后暴露一个方法调用state的方法
   ```dart
   class CheckBoxGroup extends StatefulWidget {
    ...

    CheckBoxGroupState _state;

    ...

    String getValue() {
        return _state.getCheckedValue();
    }

   }

   ```

   注意点
   因为当组件不显示时，会调用dispose方法，如果需要的资源被释放了，就会引起null错误，如果不释放，就会占用内存。

2. 好的方法
   应该在组件显示时，在onChage时通过callback方法，将状态保存到父类变量中。
