# 显示或者隐藏button

通过button的visble属性来设置
visible : lower(User().Email) = lower("aaa@hotmail.com")

visible : 画面.input1.txt != ""

visible :lookUp(datasource )

# Button 控件功能

选择 Button 控件，用户会发起一系列操作或行为，从而更改应用的状态：

- 更改显示的屏幕：Back 和 Navigate 函数。
- 控制 信号：Enable 和 Disable 函数。
- 刷新、更新或删除 数据源中的项目：Refresh、Update、UpdateIf、Patch、Remove、RemoveIf 函数。
- 更新 上下文变量：UpdateContext 函数。
- 创建、更新或删除 集合中的项：Collect、Clear、ClearCollect 函数。
- 由于这些函数可更改应用的状态，因此无法自动重新计算。 您可以在 OnSelect、OnVisible、OnHidden 和其他 On... 属性的公式（称为行为公式）中使用这些函数。

希望在 Label 控件中用红色显示小于零的值，用黑色显示大于等于零的值。 所以，可以将这个控件的 Color 属性设置为以下公式

`If( Value(TextBox1.Text) >= 0, Color.Black, Color.Red )`