# AppBar

![](img\2020-11-12-12-55-48.png)

## AppBar常用属性

* leading → Widget - 在标题前面显示的一个控件，在首页通常显示应用的 logo；在其他界面通常显示为返回按钮。
* title → Widget - Toolbar 中主要内容，通常显示为当前界面的标题文字。
* actions → List - 一个 Widget 列表，代表 Toolbar 中所显示的菜单，对于常用的菜单，通常使用 IconButton 来表示；对于不常用的菜单通常使用 PopupMenuButton 来显示为三个点，点击后弹出二级菜单。
* bottom → PreferredSizeWidget - 一个 AppBarBottomWidget 对象，通常是 TabBar。用来在 Toolbar 标题下面显示一个 Tab 导航栏。
* elevation → double - 控件的 z 坐标顺序，默认值为 4，对于可滚动的 SliverAppBar，当 SliverAppBar 和内容同级的时候，该值为 0， 当内容滚动 SliverAppBar 变为 Toolbar 的时候，修改 elevation 的值。
* flexibleSpace → Widget - 一个显示在 AppBar 下方的控件，高度和 AppBar 高度一样，可以实现一些特殊的效果，该属性通常在 SliverAppBar 中使用。
* backgroundColor → Color - Appbar 的颜色，默认值为 ThemeData.primaryColor。改值通常和下面的三个属性一起使用。
* brightness → Brightness - Appbar 的亮度，有白色和黑色两种主题，默认值为 ThemeData.primaryColorBrightness。
* iconTheme → IconThemeData - Appbar 上图标的颜色、透明度、和尺寸信息。默认值为 ThemeData.primaryIconTheme。
* textTheme → TextTheme - Appbar 上的文字样式。
centerTitle → bool - 标题是否居中显示，默认值根据不同的操作系统，显示方式不一样。
* toolbarOpacity → double

### leading

leading 属性是一个 Widget 类型
AppBar内部已经自动实现了 AppBar 的 leading 与 Drawer 抽屉栏的关联：

```dart
Widget leading = widget.leading;    if (leading == null && widget.automaticallyImplyLeading) {      if (hasDrawer) {
        leading = IconButton(
          icon: const Icon(Icons.menu),
          onPressed: _handleDrawerButton,
          tooltip: MaterialLocalizations.of(context).openAppDrawerTooltip,
        );
      } else {        if (canPop)
          leading = useCloseButton ? const CloseButton() : const BackButton();
      }
    }    if (leading != null) {
      leading = ConstrainedBox(
        constraints: const BoxConstraints.tightFor(width: _kLeadingWidth),
        child: leading,
      );
    }

```

leading 如果 为空并且 automaticallyImplyLeading 属性为true，那么就会自动推断出 leading 的 Widget 的类型和用途，另外如果当前 Scaffold 中存在 Drawer ，则会自动创建一个 IconButton 作为leading  使用，同时它的点击事件中处理了与 Drawer 抽屉栏的关联事件，无需开发者再处理。
也就是用了Drawer就不会有back按钮。
可以自己将leading设置成一个装了两个图标（回退，Drawer）的容器，然后自己处理点击事件

* 回退 Navigator.of(context).pop()
* Drawer _scaffoldKey.currentState.openDrawer();

### actions

如下图就是appbar的右边部分，通常会放置IconButton，PopupMenuButton。

![](img\2020-11-26-18-44-11.png)

```
return AppBar(
  title: Text(
    title,
    style: TextStyle(color: Theme.of(context).colorScheme.onPrimary),
  ),
  centerTitle: true,
  actions: <Widget>[
    createShopSearchIcon(context),
    createShowPassportIcon(),
    createPopUpMenu(context),
  ]);

Widget createShopSearchIcon(BuildContext context) {
    return IconButton(
        icon: new Icon(Icons.search),
        tooltip: '特典検索',
        onPressed: () {
          Navigator.pushNamed(context, MspPages.page_shop_search);
        });
  }

Widget createPopUpMenu(BuildContext context) {
    AppBL appBL = Provider.of<AppBL>(context, listen: false);
    // 隐藏的菜单
    return PopupMenuButton<String>(
      itemBuilder: (BuildContext context) => <PopupMenuItem<String>>[
        selectView(Icons.message, 'sky1', 'sky1'),
        selectView(Icons.group_add, 'sky2', 'sky2'),
        selectView(Icons.cast_connected, 'sky3', 'sky3'),
      ],
      onSelected: (String action) {
        // 点击选项的时候
        switch (action) {
          case 'sky1':
            appBL.changeTheme(MspThemeEnum.sky1);
            // appBL.changeTheme(MspThemeEnum.sky);
            break;
          case 'sky2':
            appBL.changeTheme(MspThemeEnum.sky2);
            break;
          case 'sky3':
            appBL.changeTheme(MspThemeEnum.sky3);
            break;
          default:
            appBL.changeTheme(MspThemeEnum.sky1);
            break;
        }
      },
    );
  }

  // 返回每个隐藏的菜单项
  selectView(IconData icon, String text, String id) {
    return new PopupMenuItem<String>(
        value: id,
        child: new Row(
          mainAxisAlignment: MainAxisAlignment.spaceEvenly,
          children: <Widget>[
            new Icon(icon, color: Colors.blue),
            new Text(text),
          ],
        ));
  }

```