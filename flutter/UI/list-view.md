# ListView

是一个性能比较好（和SingleChildScrollView比）的支持滚动的组件，是因为它支持基于Sliver的延迟实例化模型。

## 可以利用属性 List\<Widget\> children，来设置要显示的每一行内容
其实就是纵向排列的每一行，一般会用到Row，ListTile,Card

示例 
```
Widget build(BuildContext context) {
    return ListView(
    children:  _buildList(),
    );
}


Iterable<Widget> _buildList() {
    List<Widget> list = List();
    list.add(MspSpacer.spacerH10());
    /// 从内容List取出数据
    _list.forEach((element) {
      String createdTime = MspDateFormat.yyyyMMddHHmmSS(element.createdTime);
      String content = element.content;
      list.add(
              ListTile(
               title: Text(
                 element.title,
                 overflow: TextOverflow.ellipsis,
               ),
               trailing: Icon(Icons.keyboard_arrow_right),
               onTap: () {
                 Navigator.of(context).pushNamed(
                     MspPages.page_notification_detail,
                     arguments: element);
      );
    });
    return list;
}
```

## 可以利用ListView.builder来追加每一行的内容
需要设置
itemCount loop循环的数
itemBuilder 一个builder函数，来构建每一行显示的组件

示例 
```dart
Widget build(BuildContext context) {
  return ListView.builder(
            itemCount: _shopList.length,
            itemBuilder: (context, idx) {
              /// 店舗名
              return  TextButton(
                          child: Text( _shopList.elementAt(idx).shopName,),
                          onPressed: () {
                                  Navigator.pushNamed(context,MspPages.page_shop_detail,arguments: _shopList.elementAt(idx).id,);
                                },
              );
            },
        );
}
```

## 下拉刷新
只要在ListView外包装一层RefreshIndicator就可以了
需要设置一个下拉处理的异步函数final RefreshCallback onRefresh
示例 
```dart
Widget build(BuildContext context) {
    return RefreshIndicator(
        /// 下拉刷新函数
        onRefresh: () async {
            MspLogger.logger.d("notification on refresh");
            _req = PageReq();
            /// 重新获取最新数据
            _fetchData();
        },
        /// ListView组件
        child: ListView(children:  _buildList(),),
    );
}

```

## 上拉翻页
实现上拉翻页要用ScrollController，在Controller中加入Listener用来监听是否已经拖到屏幕边界，然后将controller加入到Listview中。

```
 /// 1. 创建ScrollController实例
 ScrollController _scrollController = ScrollController();

/// 2. 在initState中addListener监听是否已经拖到屏幕边界
  @override
  void initState() {

      _scrollController.addListener(() {
      if (_scrollController.position.pixels ==
          _scrollController.position.maxScrollExtent) {
        print('一番下に達 [${_req.pageIndex}] [${_res.totalPage}]');
        /// 判断是否到了最后一页
        if (_req.pageIndex < _res.totalPage) {
          _req.pageIndex++;
          _fetchData();
        } else {
          // TODO display no more data message
          Scaffold.of(context).showSnackBar(const SnackBar(
            content: const Text('no more data'),
          ));
        }
      }
    });
    super.initState();

  ｝

/// 3. 在listview中设定ScrollController

 Widget build(BuildContext context) {
    return Scaffold(
      appBar: MspAppBar(title: 'お知らせ一覧'),
      body: ListView(
            controller: _scrollController,
            children:  _buildList(),
        ),
    );
  }  

```

## ListTile

![](img\2020-11-02-16-13-33.png)

## 实现左滑删除

只要将列表中的每个项外面包装一层Dismissible Widget。然后实现以下函数
* **ConfirmDismissCallback confirmDismiss** 
  typedef ConfirmDismissCallback = Future\<bool\> Function(DismissDirection direction)
  这个CallBack函数应该弹出一个对话框，提示用户确认是否删除，如果点击[确认]按钮，就会返回true，然后会接着调用onDismissed函数，如果点击[取消]按钮，就会返回false，然后不会调用onDismissed函数。

* **DismissDirectionCallback onDismissed**
  typedef DismissDirectionCallback = void Function(DismissDirection direction);
  这个函数就做具体的删除动作，例如向服务器发出删除请求。。。

实现左滑删除前代码
```
new ListView.builder(
  itemCount: items.length,
  itemBuilder: (context, index) {
    return new ListTile(title: new Text('${items[index]}'));
  },
);
```

实现左滑删除后代码
```dart
new ListView.builder(
  itemCount: items.length,
  itemBuilder: (context, index) {
    return new Dismissible(
        /// 必须填写，可以用item的id，必须唯一
        key: new Key(items[index].id),
        /// confirmDismiss call back function
        confirmDismiss:  (DismissDirection direction) async {
            /// 弹出确认对话框
            return UIUtil.showDeleteConfirmDialog(context, "お知らせ削除してもよろしいでしょうか。");
        },
        /// 删除函数，做具体删除的工作
        onDismissed: (direction) {
            /// 删除选中的item数据
            items.removeAt(index);

            // 显示删除完后提示消息框
            Scaffold.of(context).showSnackBar(
                new SnackBar(content: new Text("$item dismissed")));
        },
        /// 显示的item
        child: new ListTile(title: new Text('${items[index]}')),
    );
  },
);
```
如果出现以下错误
`A dismissed Dismissible widget is still part of the tree.`
是因为你没有删除掉数据，或者没有向服务器发送删除请求，刷新重新又获得了已经删除的数据。Dismissible组件是通过key来判断唯一性的。

