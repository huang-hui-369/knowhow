# 获取依赖库文件
在项目根目录下运行

    `flutter packages get`

或

    `flutter pub get`

# Rebuild your app
`flutter run`    

# Flutter 升级dart sdk version
`flutter upgrade`

# 更新/回退到指定版本
`flutter version 1.22.0-1.0.pre`

1. 在git hub上的tag里找到想要的版本的commit的hash码
   https://github.com/flutter/flutter/tags

2. 确定flutter安装目录
    `flutter docter -v`

3. 进入flutter安装目录，执行
   `git reset --hard hash码`

4. 执行
    `flutter doctor`

# 切换channel
flutter channel stable（beta/master/dev）

# 资源文件修改了，一定要重新安装。