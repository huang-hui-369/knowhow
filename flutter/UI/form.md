# form
如果超出屏幕长度，想要滚动的话，一定要用SingleChildScrollView，因为SingleChildScrollView的更新buandary是所有组件，即使向下滚动上面的组件不显示了，也会在内存render tree中，不会把不显示的组件deactivate，disopose掉，所以组件的状态也是保存的。
<font color='red'>一定不能用ListView</font>,因为ListView考虑到性能的问题超出屏幕的范围的组件会被deactivate，disopose掉，状态也会丢失，再次显示时，会重新mount，initState,所以入力的数据也会丢失，除非在onChanged中调用setState(),来保存入力数据，这样的话界面会被多次重新描绘。针对于statusFullWidget 只有调用了setState()状态才会被保存。

# textfield

除了初始化时，textfield不能程序设值，只能用户输入。

可以通过 TextEditingController(text: widget.dateTimeFormatter(_selectedDateTime));
绑定一个变量，以后可以通过变量获得用户输入值。


 keyboardType: TextInputType.emailAddress