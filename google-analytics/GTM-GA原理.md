# web gtm 原理

![](img\2021-06-16-15-53-34.png)

实际上就是将给server 发送用户数据的功能单独抽出一个js文件GTM.js，
然后在web site的页面中引入GTM.js，

如果想增加更改发送的用户数据，可以通过GTM site来追加修改tag，这样的话就间接的修改了GTM.js文件，
而不需要修改 web site 页面的代码。

# android，ios 的gtm 原理

和web gtm 原理是一致的，事先需要给app引入 google analytics sdk
app启动时，或者定时向GA server获取tag设置数据，就是gtm的container的json文件，
然后在满足trigger的时候向GA server发送数据。

如果想增加更改发送的用户数据，可以通过GTM site来追加修改tag，这样的话就间接的修改了gtm的container的json文件，google analytics sdk会根据最新的container的json文件，来追加修改发送的数据，而不需要修改 app的代码。


