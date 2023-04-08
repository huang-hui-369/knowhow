# Activity生命周期

- onCreate

  创建的时候调用

- onStart

  由不可见变可见的时候调用

- onResume

  追备好和用户交互的时候调用

- onPause

  系统准备去启动或恢复另一个Activity时调用，会将一些消耗cpu的资源释放掉，或者保存关键数据，一定要快。

- onStop

  Activity完全不可见的时候调用，调用对话框的Activity时onPause会被调用，onStop不会被调用。

- onDestroy

  在被销毁之前调用

- onRestart

  在停止转态变为运行状态之前调用。

onStart -- onStop是可见周期

onResume -- onPause 是可交互周期（运行状态）

![image-20220630191405988](.\008Activity的生命周期.assets\image-20220630191405988.png)



